1. # Datadog APM Service 연동

   ## 지침

   1. Deployment 설정

      ```
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        labels:
          tags.datadoghq.com/env: <env>
          tags.datadoghq.com/service: <service>
          tags.datadoghq.com/version: <version>
      spec:
        template:
          metadata:
            labels:
              tags.datadoghq.com/env: <env>
              tags.datadoghq.com/service: <service>
              tags.datadoghq.com/version: <version>
              admission.datadoghq.com/enabled: "true"
            annotations:
              admission.datadoghq.com/js-lib.version: v3.18.0
          spec:
            containers:
              - name: <CONTAINER_NAME>
                image: <CONTAINER_IMAGE>/<TAG>
                env:
                  - name: DD_AGENT_HOST
                    valueFrom:
                      fieldRef:
                        fieldPath: status.hostIP
                  - name: DD_LOGS_INJECTION
                    value: "true"
                  - name: DD_ENV
                    valueFrom:
                      fieldRef:
                        fieldPath: metadata.labels['tags.datadoghq.com/env']
                  - name: DD_SERVICE
                    valueFrom:
                      fieldRef:
                        fieldPath: metadata.labels['tags.datadoghq.com/service']
                  - name: DD_VERSION
                    valueFrom:
                      fieldRef:
                        fieldPath: metadata.labels['tags.datadoghq.com/version']
      ```

   2. tracer 설정

      1) tarcer.ts 작성

      ```
      import tracer from 'dd-trace';
      
      tracer
        .init({ logInjection: true })
        .use('http', {
          blocklist: ['/health_check'], // health_ckech 제외
        })
        .use('express', { // apm 사용시 검색에 필요한 태그 설정
          hooks: {
            request: (span, req: any) => {
              const user = req.user;
              if (user) {
                span.setTag('user.id', user.id);
                span.setTag('user.name', user.name);
                span.setTag('user.email', user.email);
              }
            },
          },
        });
      
      export default tracer;
      ```

      2) main.ts 에 tracer import 추가

      ```
      import './tracer';
      ```

   3. Logger 작성

      ```
      const tracer = require('dd-trace');
      const formats = require('dd-trace/ext/formats');
      
      class Logger {
          log(level, message) {
              const span = tracer.scope().active();
              const time = new Date().toISOString();
              const record = { time, level, message };
      
              if (span) {
                  tracer.inject(span.context(), formats.LOG, record);
              }
      
              console.log(JSON.stringify(record));
          }
      }
      
      module.exports = Logger;
      ```

   ## 관련 문서

   1. APM 연동
      https://app.datadoghq.com/apm/service-setup?architecture=container-based&collection=Helm Chart (Recommended)&environment=kubernetes&language=node&profiling=false
   2. tracer 설정
      https://docs.datadoghq.com/tracing/trace_collection/custom_instrumentation/nodejs/?tab=component 
   3. Logger 작성
      https://docs.datadoghq.com/tracing/other_telemetry/connect_logs_and_traces/nodejs/ 