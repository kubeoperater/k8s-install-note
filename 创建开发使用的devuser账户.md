``` bash
cfssl gencert -ca=../ca.pem -ca-key=../ca-key.pem -config=../ca-config.json -profile=example devuser-csr.json | cfssljson -bare devuser
kubectl config set-cluster kubernetes \
--certificate-authority=/etc/kubernetes/ssl/ca.pem \
--embed-certs=true \
--server=${KUBE_APISERVER} \
--kubeconfig=devuser.kubeconfig

kubectl config set-credentials devuser \
--client-certificate=devuser.pem \
--client-key=devuser-key.pem \
--embed-certs=true \
--kubeconfig=devuser.kubeconfig


kubectl config set-context kubernetes \
--cluster=kubernetes \
--user=devuser \
--namespace=kube-system \
--kubeconfig=devuser.kubeconfig


kubectl config use-context kubernetes --kubeconfig=devuser.kubeconfig


kubectl create clusterrolebinding  devuser-clusteradmin-bindding --clusterrole cluster-admin --user devuser
```
