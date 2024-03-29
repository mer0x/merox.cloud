#### wikijs-config.yaml
``` yaml linenums="1"
---
apiVersion: v1
kind: Service
metadata:
  name: mariadb
  namespace: wikijs
spec:
  selector:
    app: mariadb
  ports:
  - name: mariadb
    protocol: TCP
    port: 3306
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mariadb
  namespace: wikijs
  labels:
    app: mariadb
spec:
  selector:
    matchLabels:
      app: mariadb
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      containers:
      - name: mariadb
        image: mariadb:latest
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mariadb-secret
              key: ROOT_PASSWORD
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              name: mariadb-secret
              key: DATABASE
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: mariadb-secret
              key: USER
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mariadb-secret
              key: PASSWORD
        - name: MARIADB_ROOT_HOST
          value: "%"
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: wikijs-db
          mountPath: /var/lib/mysql
      volumes:
      - name: wikijs-db
        persistentVolumeClaim:
          claimName: wikijs-pv-claim
```
#### wikijs-deployment.yaml
```yaml linenums="1"
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wikijs
  namespace: wikijs
  labels:
    app: wikijs
spec:
  selector:
    matchLabels:
      app: wikijs
  template:
    metadata:
      labels:
        app: wikijs
    spec:
      containers:
      - name: wikijs
        image: requarks/wiki:latest
        imagePullPolicy: Always
        env:
        - name: DB_TYPE
          value: "mariadb"
        - name: DB_HOST
          value: "mariadb"
        - name: DB_PORT
          value: "3306"
        - name: DB_NAME
          valueFrom:
            secretKeyRef:
              name: mariadb-secret
              key: DATABASE
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: mariadb-secret
              key: USER
        - name: DB_PASS
          valueFrom:
            secretKeyRef:
              name: mariadb-secret
              key: PASSWORD
        ports:
        - containerPort: 3000
          name: http
```
#### wikijs-ingress.yaml

```yaml linenums="1"
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: wikijs
  namespace: wikijs
  annotations: 
    kubernetes.io/ingress.class: traefik-external
spec:
  entryPoints:
    - websecure
  routes:
    - match: Host(`www.merox.cloud`) # change to your domain
      kind: Rule
      services:
        - name: wikijs
          port: 3000
    - match: Host(`merox.cloud`) # change to your domain
      kind: Rule
      services:
        - name: wikijs
          port: 3000
      middlewares:
        - name: wikijs-middleware
  tls:
    secretName: mer0x39-tls # change to your cert name
```

#### wikijs-middleware.yaml

```yaml linenums="1"

apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: wikijs-middleware
  namespace: wikijs
spec:
  headers:
    browserXssFilter: true
    contentTypeNosniff: true
    forceSTSHeader: true
    stsIncludeSubdomains: true
    stsPreload: true
    stsSeconds: 15552000
    customFrameOptionsValue: SAMEORIGIN
    customRequestHeaders:
      X-Forwarded-Proto: https
```


#### wikijs-pvc.yaml

```yaml linenums="1"
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: wikijs-pv-claim
  namespace: wikijs
spec:
  storageClassName: longhorn
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```
#### wikijs-secret.yaml

```yaml linenums="1"
apiVersion: v1
kind: Secret
metadata:
  name: mariadb-secret
  namespace: wikijs
type: Opaque
data:
  ROOT: cm9vdA==
  ROOT_PASSWORD: VGhpdqw#@423yISE1Nw==
  DATABASE: zdzd22lrsaWpz
  USER: zdzd22lrsaWpz
  PASSWORD: VGhpdqw#@423yISE1Nw==
```

#### wikijs-service.yaml

```yaml linenums="1"
apiVersion: v1
kind: Service
metadata:
  name: "wikijs"
  namespace: wikijs
spec:
  type: ClusterIP
  ports:
    - name: http
      port: 3000
  selector:
    app: "wikijs"
```