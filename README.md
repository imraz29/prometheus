# prometheus
#alerting-rules
first search for prometheus configmaps in the namespace ex-
> kubectl get cm -n prometheus

edit the prometheus-alertmanager.yml 
example > 

        apiVersion: v1
        data:
          alertmanager.yml: |
            global:
              resolve_timeout: 1m
              smtp_smarthost: 'smtp.office365.com:587'
              smtp_from: 'imraz@pixalive.me'
              smtp_auth_username: 'imraz@pixalive.me'
              smtp_auth_password: 'Sheeraz@7619'
              smtp_require_tls: true
              
            route:
              receiver: 'outlook-notifications'
        
            receivers:
              - name: 'outlook-notifications'
                email_configs:
                - to: 'kabeer@pixalive.me,kiran@pixalive.me,imraz@pixalive.me,prashanth@pixalive.me,rajasekar@pixalive.me,sathish@pixalive.me'
                  send_resolved: true
                  from: 'kiran@pixalive.me'
                  smarthost: 'smtp.office365.com:587'
                  auth_username: 'kiran@pixalive.me'
                  auth_identity: 'kiran@pixalive.me'
                  auth_password: 'Pascalpass@167^'
                  require_tls: true
                  headers:
                    Subject: '[{{ .Status | toUpper }}] {{ .CommonLabels.alertname }} for {{ .CommonLabels.job }}'
                  html: >-
                    <html>
                    <body>
                      {{ range .Alerts -}}
                      <h3>Alert: {{ .Annotations.title }}{{ if .Labels.severity }} - <b>{{ .Labels.severity }}</b>{{ end }}</h3>
                      <p>{{ .Annotations.description }}</p>
                      <ul>
                        {{ range .Labels.SortedPairs }}
                        <li><b>{{ .Name }}:</b> {{ .Value }}</li>
                        {{ end }}
                      </ul>

edit prometheus-server.yml file 
example-
apiVersion: v1
data:
  alerting_rules.yml: |
    groups:

    - name: KubeStateAlerts

      rules:

        - alert: KubernetesNodeReady
          expr: 'kube_node_status_condition{condition="Ready",status="true"} == 0'
          for: 10m
          labels:
            severity: critical
          annotations:
            summary: Kubernetes Node ready (instance {{ $labels.instance }})
            description: "Node {{ $labels.node }} has been unready for a long time\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

        - alert: KubernetesMemoryPressure
          expr: 'kube_node_status_condition{condition="MemoryPressure",status="true"} == 1'
          for: 2m
          labels:
            severity: critical
          annotations:
            summary: Kubernetes memory pressure (instance {{ $labels.instance }})
            description: "{{ $labels.node }} has MemoryPressure condition\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

        - alert: KubernetesDiskPressure
          expr: 'kube_node_status_condition{condition="DiskPressure",status="true"} == 1'
          for: 2m
          labels:
            severity: critical
          annotations:
            summary: Kubernetes disk pressure (instance {{ $labels.instance }})
            description: "{{ $labels.node }} has DiskPressure condition\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

        - alert: KubernetesNetworkUnavailable
          expr: 'kube_node_status_condition{condition="NetworkUnavailable",status="true"} == 1'

then run command to apply :
> kubectl apply -f prometheus-server prometheus-alertmanager
