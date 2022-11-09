# ServerClient


### To install:
`helm install --wait full-coral ./serverclient`

### To uninstall:
`helm uninstall full-coral`


### Install with ClusterIP:
```
helm install --set service.type=ClusterIP full-coral .
```