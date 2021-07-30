
withCredentials([string(credentialsId: 'cypressProjectId', variable: 'cypressProjectId'),string(credentialsId: 'cypressRecordKey', variable: 'cypressRecordKey'),string(credentialsId: 'PERCY_TOKEN', variable: 'PERCY_TOKEN')]){
    String key = cypressParam.substring(cypressParam.indexOf("--tags=") + 7, cypressParam.indexOf("--wlp")).replace("'","")
    if (!(cypressParam.contains("--parallel-group"))) {
        wlp = cypressParam.substring(cypressParam.indexOf("--wlp=") + 6, cypressParam.length())
    } else {
        wlp = cypressParam.substring(cypressParam.indexOf("--wlp=") + 6, cypressParam.indexOf("--parallel-group"))
        group = cypressParam.substring(cypressParam.indexOf("--parallel-group=") + 17, cypressParam.length())
        parallelGroup = []
        parallelGroup = group.split(",")
    }

    String finalTag;
    String finalCommand;

    if ((cypressParam.contains("--parallel-group"))) {
        String[] segments = key.split(" ");
        
        def tagList = [];
        segments.each {
            if ((it.contains("@"))) {
            tagList << it
            }
        }
        for (int i = 0; i < tagList.size(); i++) {
            finalCommand = "auto:server --tags='" + tagList[i] + "' --wlp=${wlp} --parallel-group=${parallelGroup[i]}"
            if ((tagList[i].contains("login"))) {
                automateTest("${finalCommand}","${wlp}")
            } else {
                def parallelStages = [:]
                for (int z = 0; z < 3; z++) {
                    def parallelName = "Test ${wlp} - ${tagList[i]} - " + z
                    parallelStages.put(parallelName, prepareBuildStage("${finalCommand}","${wlp}"))
                }
                parallel(parallelStages)
            }
        }
    } else {
        automateTest("${cypressParam}","${wlp}")
    }
}

def automateTest(String buildName, String wlpName) {
    node('build') {
        try {
            stage("Test ${wlpName}") {
                withEnv(["buildName=${buildName}"]){
                    git branch: "master",
                        credentialsId: 'yusuftravlr-github',
                        url: 'https://github.com/ysmuhammad/nodejs'
                    sh '''#!/usr/bin/env bash
                        export tagName=$(git tag --sort=committerdate | tail -1)
                        echo "tagName: ${tagName}"
                    '''
                    echo "tagName groovy: ${env.tagName}"
                    checkout(
                        [
                            $class: 'GitSCM',
                            branches: [[name: "refs/tags/${env.tagName}"]],
                            userRemoteConfigs: [
                                [credentialsId: 'yusuftravlr-github', url: 'https://github.com/ysmuhammad/nodejs']
                            ]
                        ]
                    )
                    AUTOMATE_TEST = sh (
                    script: '''#!/usr/bin/env bash
                            set -x
                            echo "Running Test $buildName ..."
                            ls -l
                            ''',
                    returnStatus: true
                    )
                    if ("${AUTOMATE_TEST}" != "0") {
                        error "Test FAILED! Please see the log above ^^^"
                    }
                }
            }
        } finally {
            deleteDir()
        }
    }
}

def prepareBuildStage(String buildName, String wlpName){
    return {
        node('build') {
            try {
                stage("Test ${wlpName}") {
                    withEnv(["buildName=${buildName}"]){
                        git branch: "master",
                            credentialsId: 'yusuftravlr-github',
                            url: 'https://github.com/ysmuhammad/nodejs'
                        sh '''#!/usr/bin/env bash
                            export tagName=$(git tag --sort=committerdate | tail -1)
                        '''
                        checkout(
                            [
                                $class: 'GitSCM',
                                branches: [[name: "refs/tags/${env.tagName}"]],
                                userRemoteConfigs: [
                                    [credentialsId: 'yusuftravlr-github', url: 'https://github.com/ysmuhammad/nodejs']
                                ]
                            ]
                        )
                        AUTOMATE_TEST = sh (
                        script: '''#!/usr/bin/env bash
                                set -x
                                echo "Running Test $buildName ..."
                                ls -l
                                ''',
                        returnStatus: true
                        )
                        if ("${AUTOMATE_TEST}" != "0") {
                            error "Test FAILED! Please see the log above ^^^"
                        }
                    }
                }
            } finally {
                deleteDir()
            }
        }
    }
}
