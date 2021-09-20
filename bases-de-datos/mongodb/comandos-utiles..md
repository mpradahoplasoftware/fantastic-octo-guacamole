# Comandos útiles.

## ¿Cómo borrar un statefulset?

* oc delete sts --cascade=false replica-set-pro 
* oc apply -f PRO\mongo-pro\TLS-replica-set-pro\replica-set-tirea-replicaSetHorizons-scram.yaml 
* oc rollout restart sts replica-set-pro

## ¿Cómo borrar el un miembro de un replicaset?

* oc delete pods replica-set-pro-2 --grace-period=0 --force=true  
* oc delete pvc data-replica-set-pro-2 --grace-period=0 --force=true 
* oc delete pods replica-set-pro-2



