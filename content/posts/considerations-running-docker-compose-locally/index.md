+++
title = 'Considerations for Running Docker-Compose Locally'
date = 2024-04-10T15:37:06-05:00
featured_image = 'powershell.jpg'
draft = false
tags = ["docker", "ci-cd"]
+++


# Run tests locally

In order to test, you can spin up the docker containers locally.
There are multiple things that you need to do to avoid confusion during testing.

The code that is tested, gets copied over to a docker image. 
This means that when you make a change, you need to take care of things such as:

- stop running containers
- remove containers
- remove volumes that have stale data
- remove images for UI and backend API
- remove dangling images
- spin up new services with docker compose
- copy tests results into specific directory

After doing this a couple of times
(and forgetting to do them) it was time for automation.


Here is a powershell script that automated things and freed up my time to do more important things.

```powershell
#
# Helper powershell script to automate testing on local machine.
#
Function Get-TypeOfTest {
  $type=Read-Host "
    1 - All Tests
    2 - Smoke Tests
    Please choose"
  Switch ($type){
    1 {$choice="all"}
    2 {$choice="smoke"}
  }
  return $choice
}

$testType=Get-TypeOfTest

# Use docker compose file according to user's selection of tests
$dockerComposeFile = ".\docker-compose-e2e-local-$testType-tests.yml"

# stop and remove containers defined in the compose file
docker compose -f .\docker-compose-e2e-base.yml -f $dockerComposeFile down

# remove images so they can be rebuilt
$api_image=$(docker images --format "{{.Repository}}:{{.Tag}}"|findstr "api_for_ci_e2e_test")
$ui_image =$(docker images --format "{{.Repository}}:{{.Tag}}"|findstr "ui_for_ci_e2e_tests")
if ($api_image){  docker rmi $api_image }
if ($ui_image){  docker rmi $ui_image }

# remove volumes
docker volume prune -f

# spin up containers
docker compose -f .\docker-compose-e2e-base.yml  -f $dockerComposeFile up --abort-on-container-exit

# copy screenshots to local folder and open it
docker cp cypress:/e2e/cypress/screenshots ./results/cypress_screenshots
docker cp cypress:/e2e/test_results ./results/test_results

# open folder with tests results
explorer .\results
```
