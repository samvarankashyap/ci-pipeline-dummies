node() {
    stage ('build'){
        openshiftBuild(buildConfig: 'fedora', showBuildLogs: 'true')
    }
    
    stage ('deploy'){
    // make sure you have twinecreds with keys twine_username, twine_password set for uploading pypi packages
        openshiftCreateResource(jsonyaml: '{"kind":"Pod","apiVersion":"v1","metadata":{"name":"hello-openshift","creationTimestamp":null,"labels":{"name":"hello-openshift"}},"spec":{"containers":[{"name":"hello-openshift","image":"172.30.1.1:5000/minishift-demo/fedora","ports":[{"containerPort":8080,"protocol":"TCP"}],"resources":{},"volumeMounts":[{"name":"tmp","mountPath":"/tmp"}],"env":[{"name":"TWINE_USERNAME","valueFrom":{"configMapKeyRef":{"name":"twinecreds","key":"twine_username"}}},{"name":"TWINE_PASSWORD","valueFrom":{"configMapKeyRef":{"name":"twinecreds","key":"twine_password"}}}],"terminationMessagePath":"/dev/termination-log","imagePullPolicy":"IfNotPresent","capabilities":{},"securityContext":{"capabilities":{},"privileged":false}}],"volumes":[{"name":"tmp","emptyDir":{}}],"restartPolicy":"Always","dnsPolicy":"ClusterFirst","serviceAccount":""},"status":{}}')
    }

    stage('wait'){
        echo 'Waiting 30 seconds for deployment to complete prior starting smoke testing' 
        sleep 30 
    }
    stage('install_deps'){
        openshiftExec(pod: 'hello-openshift', container: 'hello-openshift', namespace: 'minishift-demo', command: 'dnf', arguments: ['install','python-pip','git', '-y'])
        openshiftExec(pod: 'hello-openshift', container: 'hello-openshift', namespace: 'minishift-demo', command: 'pip', arguments: ['install','twine'])
    }
    stage('clone_src'){
        openshiftExec(pod: 'hello-openshift', container: 'hello-openshift', namespace: 'minishift-demo', command: 'pwd')
        openshiftExec(pod: 'hello-openshift', container: 'hello-openshift', namespace: 'minishift-demo', command: 'git', arguments: ['clone','https://github.com/samvarankashyap/gummybears'])
        openshiftExec(pod: 'hello-openshift', container: 'hello-openshift', namespace: 'minishift-demo', command: 'ls')
    }
    stage('install_src'){
        openshiftExec(pod: 'hello-openshift', container: 'hello-openshift', namespace: 'minishift-demo', command: 'pwd')
        openshiftExec(pod: 'hello-openshift', container: 'hello-openshift', namespace: 'minishift-demo', command: 'pip', arguments: ['install', 'file:///gummybears'])
    }

    stage ('test'){
        openshiftExec(pod: 'hello-openshift', container: 'hello-openshift', namespace: 'minishift-demo', command: 'gummybears')
        def response = openshiftExec(pod: 'hello-openshift', container: 'hello-openshift', namespace: 'minishift-demo', command: 'python', arguments: ['./gummybears/test_gummybears.py'])
            if ( response.error == "" && response.failure == "" ) {
                println('Stdout: '+response.stdout.trim())
                println('Stderr: '+response.stderr.trim())
        }
    }

    stage ('cleanup'){
        openshiftDeleteResourceByJsonYaml(jsonyaml: '{"kind":"Pod","apiVersion":"v1","metadata":{"name":"hello-openshift","creationTimestamp":null,"labels":{"name":"hello-openshift"}},"spec":{"containers":[{"name":"hello-openshift","image":"172.30.1.1:5000/minishift-demo/fedora","ports":[{"containerPort":8080,"protocol":"TCP"}],"resources":{},"volumeMounts":[{"name":"tmp","mountPath":"/tmp"}],"env":[{"name":"TWINE_USERNAME","valueFrom":{"configMapKeyRef":{"name":"twinecreds","key":"twine_username"}}},{"name":"TWINE_PASSWORD","valueFrom":{"configMapKeyRef":{"name":"twinecreds","key":"twine_password"}}}],"terminationMessagePath":"/dev/termination-log","imagePullPolicy":"IfNotPresent","capabilities":{},"securityContext":{"capabilities":{},"privileged":false}}],"volumes":[{"name":"tmp","emptyDir":{}}],"restartPolicy":"Always","dnsPolicy":"ClusterFirst","serviceAccount":""},"status":{}}')

    }
}
