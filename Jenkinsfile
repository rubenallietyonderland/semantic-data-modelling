pipeline {
    agent {
        label 'bi'
    }

    triggers {
        // Only works for Multibranch Pipeline if webhooks are correctly set
        pollSCM('H/5 * * * *')
    }

    options {
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '90', numToKeepStr: '30')
        durabilityHint 'MAX_SURVIVABILITY'
        timeout(300)
        timestamps()
        ansiColor('xterm')
    }

    stages {
        stage('Hello') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'github-tmp-credentials', url: 'https://github.com/rubenallietyonderland/semantic-data-modelling.git']])
            }
        }
    }
}


// def git_repo = "git@bitbucket.org:asadventure/lakehouse-dbt-cicd.git"

// def vaultConfig = [
//     $class       : 'VaultTokenCredentialBinding',
//     credentialsId: 'hashicorp-vault-jenkins-approle-token',
//     vaultAddr    : 'https://vault.tools.yonderland.com'
// ]

// pipeline {
//     agent {
//         label 'bi'
//     }

//     options {
//         buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '90', numToKeepStr: '30')
//         durabilityHint 'MAX_SURVIVABILITY'
//         timeout(300)
//         timestamps()
//         ansiColor('xterm')
//     }

//     environment {
//         // Define other environment variables
//         SNOWFLAKE_PRIVATE_KEY_PATH = credentials('lakehouse-jenkins-dbt-cicd-private-key')
//         AIRFLOW_HOME = "${WORKSPACE}/airflow"
//         AIRFLOW_IMAGE = 'apache/airflow:2.10.1'
//     }

//     stages {
//         stage('Determine Branch') {
//             steps {
//                 script {
//                     def branchName = env.BRANCH_NAME
//                     env.BITBUCKET_PR_DESTINATION_BRANCH = branchName

//                     echo "Building branch: ${branchName}"

//                     if (branchName ==~ /^feature\/LH-.*/ || branchName == 'master') {
//                         echo "Branch '${branchName}' matches the criteria."
//                     } else {
//                         error "Branch '${branchName}' does not match the required pattern."
//                     }
//                 }
//             }
//         }

//         stage('Get commit details') {
//             steps {
//                 script {
//                     def gitAuthorEmail = sh(script: "git --no-pager show -s --format='%ae' ${GIT_COMMIT}", returnStdout: true).trim()
//                     env.GIT_AUTHOR = gitAuthorEmail.replaceAll(/@.*$/, '').replace('.', '_').toUpperCase()
//                     echo "Author name = ${env.GIT_AUTHOR}"
//                 }
//             }
//         }

//         stage('Check Python 3.11 and Pip Version') {
//             steps {
//                 script {
//                     echo "Checking Python version:"
//                     sh 'python3.11 --version'
                    
//                     echo "Checking Pip version:"
//                     sh 'python3.11 -m pip --version'
//                 }
//             }
//         }

//         stage('Install Dependencies') {
//             steps {
//                 script {
//                     try {
//                         echo "=== 🧱 Installing dependencies ==="
//                         sh 'python3.11 -m pip install -r dbt/requirements.txt'
                        
//                         env.PATH = "/usr/local/bin:${env.PATH}"

//                         def dbtCoreInstalled = sh(script: 'python3.11 -m pip show dbt-core', returnStatus: true) == 0
//                         def dbtSnowflakeInstalled = sh(script: 'python3.11 -m pip show dbt-snowflake', returnStatus: true) == 0

//                         if (!dbtCoreInstalled || !dbtSnowflakeInstalled) {
//                             echo "dbt-core or dbt-snowflake is not installed. Installing them."
//                             sh 'python3.11 -m pip install dbt-core dbt-snowflake'
//                         }
//                     } catch (Exception e) {
//                         echo "Error during pip install: ${e}."
//                         error "pip install failed."
//                     }
//                 }
//             }
//         }

//         stage('Run Branch-Specific Logic') {
//             steps {
//                 script {
//                     env.SNOWFLAKE_PRIVATE_KEY = sh(script: "cat $SNOWFLAKE_PRIVATE_KEY_PATH", returnStdout: true).trim()

//                     // Debugging step to see which branches are available
//                     sh 'git config remote.origin.fetch "+refs/heads/*:refs/remotes/origin/*"'
//                     sh 'git branch -r'
                    
//                     env.DB_NAME = "Z_${env.GIT_AUTHOR}_DEV"
//                     echo "Setting DB_NAME to ${env.DB_NAME}"

//                     def DBT_TARGET = (env.BITBUCKET_PR_DESTINATION_BRANCH ==~ /^feature\/LH-.*/) ? 'cicd_dev' : 'cicd_prod'
//                     echo "Using DBT target: ${DBT_TARGET}"

//                     // Install necessary dependencies for dbt
//                     sh 'pip3.11 install -r dbt/requirements.txt'
//                     sh 'dbt deps --project-dir dbt/'
//                     sh "dbt debug --profiles-dir dbt/ --project-dir dbt/ --target ${DBT_TARGET}"

//                     // **Clone the LAKEHOUSE_PRD schemas to Z_USERNAME_DEV**
//                     sh "dbt run-operation clone_dev_database --profiles-dir dbt/ --project-dir dbt/ --target ${DBT_TARGET}"

//                     echo "=== 🔨 Checking for changed models ==="

//                     // Fetch changed .sql and .csv files
//                     def CHANGED_MODELS = sh(script: "git diff --name-only origin/master origin/${env.BITBUCKET_PR_DESTINATION_BRANCH} -- '*.sql' '*.csv'", returnStdout: true).trim()

//                     if (CHANGED_MODELS) {
//                         def CHANGED_SEEDS = CHANGED_MODELS.tokenize().findAll { it.startsWith('dbt/seeds/') && it.endsWith('.csv') }
//                         def CHANGED_SNAPSHOTS = CHANGED_MODELS.tokenize().findAll { it.startsWith('dbt/snapshots/') && it.endsWith('.sql') }
//                         def CHANGED_MODEL_FILES = CHANGED_MODELS.tokenize().findAll { it.startsWith('dbt/models/') && it.endsWith('.sql') }

//                         // Step 1: Run seeds if there are changes
//                         if (CHANGED_SEEDS) {
//                             echo 'Found changed seeds:'
//                             echo CHANGED_SEEDS.join('\n')

//                             def SEED_PATHS = CHANGED_SEEDS.collect { "path:" + it.replace('dbt/', '') }.join(' ')
//                             sh "dbt seed --profiles-dir dbt/ --project-dir dbt/ --target ${DBT_TARGET} --select ${SEED_PATHS}"
//                         }

//                         // Step 2: Run snapshots if there are changes
//                         if (CHANGED_SNAPSHOTS) {
//                             echo 'Found changed snapshots:'
//                             echo CHANGED_SNAPSHOTS.join('\n')

//                             def SNAPSHOT_PATHS = CHANGED_SNAPSHOTS.collect { "path:" + it.replace('dbt/', '') }.join(' ')
//                             sh "dbt snapshot --profiles-dir dbt/ --project-dir dbt/ --target ${DBT_TARGET} --select ${SNAPSHOT_PATHS}"
//                         }

//                         // Step 3: Run models if there are changes
//                         if (CHANGED_MODEL_FILES) {
//                             echo 'Found changed models:'
//                             echo CHANGED_MODEL_FILES.join('\n')

//                             def MODEL_PATHS = CHANGED_MODEL_FILES.collect { "path:" + it.replace('dbt/', '') }.join(' ')
//                             sh "dbt build --profiles-dir dbt/ --project-dir dbt/ --target ${DBT_TARGET} --select ${MODEL_PATHS}"
//                         }

//                         if (!CHANGED_SEEDS && !CHANGED_SNAPSHOTS && !CHANGED_MODEL_FILES) {
//                             echo 'No valid seed, snapshot, or model file changes detected. Exiting without running dbt build.'
//                             currentBuild.result = 'SUCCESS'
//                             return
//                         }
//                     } else {
//                         echo 'No changes detected. Exiting.'
//                         currentBuild.result = 'SUCCESS'
//                         return
//                     }
//                 }
//             }
//         }
//     }
// }
