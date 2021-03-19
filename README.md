# test-environment
### Overview:
Jenkins job that will run cypress tests for all back end services 
listed in testList.txt and all front end tests specified in the 
Jenkinsfile. 

### Build Parameters:
The Jenkins job has various parameters that will allow users to 
customize the tests that are being run.
- PROJECT_URL
    - URl of the project. 
- DOCKER_REGISTRY_DOWNLOAD_URL
    - URL used to grab docker containers 
- TESTS_TO_RUN
    - Defaults to ALL, meaning all back end tests will win
    - Change to specific service name to have only one back end service 
      run
    - ex: "wmts", "oldmar", "wfs", "sqs-stager"
    - do not include "omar" or the ""
- TEST_ENV
    - Determines which environment will be tested: omar.ossim, omar-test.ossim, etc.
    - defaultvalue is omar-test.ossim
    - replace with desired "omar.ossim" combination.
- BACK_END
    - Allows back end tests to run
    - If true, will run tests according to TESTS_TO_RUN
    - default is true
- FRONT_END
    - Allows front end tests to run
    - Will run tests according to the below parameters
    - default is true
- RUN_UI
    - allows UI test to run
    - default is true
- RUN_TLV
    - allows tlv test to run
    - default is true