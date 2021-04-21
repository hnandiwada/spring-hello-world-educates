load('.tanzu/file_sync_only.py', 'file_sync_only')

mvnw = "./mvnw"
if os.name == "nt":
  mvnw = "mvnw.bat"

#  './mvnw -Dmaven.test.skip=true -Dspring-boot.repackage.excludeDevtools=false package && rm -rf .build/* && unzip -o target/demo-*.jar -d .build/',
local_resource(
  'spring-demo-compile',
  './mvnw -Dmaven.test.skip=true -Dspring-boot.repackage.excludeDevtools=false package && ' + 
    'rm -rf target/jar-staging && ' +
    'unzip -o target/demo-*.jar -d target/jar-staging && ' +
    'rsync --delete --inplace --checksum -r target/jar-staging/ .build',
  deps=['src', 'pom.xml'])

file_sync_only("hnandiwada/spring-demo",
    ["./k8s/workload.yaml", "./k8s/images.yaml"],
    deps=['.build/'],
    live_update=[
      sync('.build/BOOT-INF/lib', '/workspace/BOOT-INF/lib'),
      sync('.build/BOOT-INF/classes', '/workspace/BOOT-INF/classes'),
      sync('.build/META-INF', '/workspace/META-INF'),
      run("#TODO: figure out how to restart the container; i.e. something like 'kill -HUP 1' that actually works")
    ],
)

k8s_kind('LatestImage', image_json_path='{.spec.latestImage}', pod_readiness='ignore')
k8s_kind('ServiceFactory', api_version='knative.acp.local/v1alpha1',
         image_json_path='{.spec.template.spec.containers[].image}')

k8s_resource(workload='spring-demo:servicefactory', port_forwards=8080,
             resource_deps=['spring-demo-compile'], 
             extra_pod_selectors=[{'app': 'spring-demo'}])

