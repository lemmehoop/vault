### Немного базы
- единая точка правды, чтобы обеспечить актуальность prod'а
- обязателен в IaC, чтобы было надежно и декларативно
- автоматизирует CI/CD
### Basic info
![[Pasted image 20250704210526.png|500]]
*pipeline* - what steps should GitLab do for your CI/CD process to complete
*.gitlab-ci.yml* - file, where *pipeline* stored as written instruction
*job* - simplest part of *pipeline*. GitLab creates new container to run each of the jobs
*stage* - tag on the job, that marks, in which order jobs have to be completed
*runner* - where the jobs are running; either machine or docker container
 ### Known syntax
https://docs.gitlab.com/ee/ci/yaml/ - all syntax documentation
- `stages` - writing the order, in which jobs should be executed (`stage` written in job)
- `variables` - variables inside pipeline; work as **ENV** inside jobs (can be overall in root or in jobs)
- `image` - specifying image in root for all jobs and rewriting it inside jobs
- `before / <none> / after script`
- `needs` - setting a job another job that needs to be done to continue executing without waiting for whole stage to complete
- `dependencies` - setting a job which dependencies are required for current job (artifacts)
- `artifacts` - setting which files from job should be saved
### Predefined variables
https://docs.gitlab.com/ee/ci/variables/predefined_variables.html
- `CI_PROJECT_DIR` - directory in container where project is copied
- `CI_COMMIT_REF_NAME` - branch where project is building
- `CI_PIPELINE_SOURCE` - how pipeline was triggered ([here is list](https://docs.gitlab.com/ee/ci/jobs/job_rules.html#ci_pipeline_source-predefined-variable))
### Example
```yaml
variables:
  DIND_IMAGE: docker:28
  DOCKER_BASE_IMAGE: python:3.12-slim
  DOCKER_REGISTRY: registry.gitlab.com/lemmehoop
  DOCKER_IMAGE: $DOCKER_REGISTRY/$CI_PROJECT_NAME:$CI_COMMIT_SHORT_SHA
  POSTGRES_DB: it_costs
  POSTGRES_USER: it_costs
  POSTGRES_PASSWORD: it_costs
  POSTGRES_ALIAS: postgres
  POSTGRES_URL: "postgresql+asyncpg://${POSTGRES_USER}:${POSTGRES_PASSWORD}@${POSTGRES_ALIAS}:5432/${POSTGRES_DB}"
  POETRY: $VENV/bin/poetry
  VENV: .venv

stages:
  - cache
  - quality
  - test
  - build
  - deploy

poetry-cache:
  stage: cache
  image: $DOCKER_BASE_IMAGE
  script:
    - python3 -m venv $VENV
    - $VENV/bin/pip install -U pip setuptools
    - $VENV/bin/pip install poetry
    - $POETRY config virtualenvs.in-project true
    - $POETRY install
  artifacts:
    paths:
      - $VENV
    expire_in: 30m

.use-cache:
  image: $DOCKER_BASE_IMAGE
  dependencies:
    - poetry-cache

ruff:
  stage: quality
  extends:
    - .use-cache
  script:
    - $POETRY run ruff check --no-fix .
    - $POETRY run ruff format --check .

mypy:
  stage: quality
  extends:
    - .use-cache
  script:
    - $POETRY run mypy .

pytest:
  stage: test
  extends:
    - .use-cache
  services:
    - name: postgres:17-alpine
      alias: $POSTGRES_ALIAS
  before_script:
    - $POETRY run alembic upgrade head
  script:
    - $POETRY run pytest --cov=src --cov-report term-missing --cov-report xml
  coverage: '/TOTAL.*\s+(\d+%)$/'

build-image:
  stage: build
  image: $DIND_IMAGE
  services:
    - $DIND_IMAGE-dind
  variables:
    DOCKER_TLS_CERTDIR: "/certs"
  script:
    - echo "$CI_REGISTRY_PASSWORD" | docker login $CI_REGISTRY -u $CI_REGISTRY_USER --password-stdin
    - docker build -t $DOCKER_IMAGE .
    - docker push $DOCKER_IMAGE
  only:
    - main

deploy-swarm:
  stage: deploy
  image: $DIND_IMAGE
  tags:
    - firstvds
  script:
    - unset DOCKER_HOST
    - echo "$CI_REGISTRY_PASSWORD" | docker login $CI_REGISTRY -u $CI_REGISTRY_USER --password-stdin
    - docker stack deploy -c docker-compose.yml it-costs --detach=false --with-registry-auth
  only:
    - main

```