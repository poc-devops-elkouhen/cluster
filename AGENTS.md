# AGENTS.md — cluster

## Rôle du dépôt

`cluster` fournit le socle Kubernetes local du POC via Vagrant, Ansible et
Packer. Il ne déploie ni GitLab, ni ArgoCD, ni les applications — uniquement
le cluster et ses add-ons réseau bas niveau.

## Structure

```
vagrant/       Vagrantfile — 1 master + 1 worker VirtualBox
ansible/       Playbooks et rôles Ansible
  roles/
    zscaler/           Certificat CA corporate
    containerd/        Runtime de conteneurs
    kubernetes/        Dépôts yum + packages kubeadm/kubelet/kubectl
    kubernetes-master/ Init kubeadm, flannel, metrics-server, local-path-provisioner
    kubernetes-node/   Join du worker au cluster
    kubernetes-platform/ Gateway API, MetalLB, Traefik, Gateway partagée
packer/        Builds d'images VM reproductibles (k8s-master, k8s-worker)
```

## Versions

Les versions des composants sont dans `ansible/group_vars/all.yml`. Elles
doivent rester synchronisées avec `platform.yml` du dépôt `control-plane` —
c'est `control-plane` qui fait autorité ; modifier `all.yml` seul est une
dérive.

## Commandes principales

```bash
make up                  # Démarrer les VMs et provisionner le cluster
make create-cluster      # Démarrer les VMs Packer et initialiser le cluster
make down                # Éteindre les VMs sans les détruire
make destroy             # Détruire les VMs
cd packer && make build  # Construire les images VM Packer
```

## Contraintes Vagrant / QEMU

- Ne pas proposer de workflow nécessitant Vagrant ou QEMU en root.
- Pour le provider QEMU, ne pas utiliser le pattern réseau point-à-point où le
  master écoute et le worker se connecte.

## Ce qu'il ne faut pas faire

- Ne pas déployer GitLab, ArgoCD ou des applications depuis ce dépôt.
- Ne pas modifier `group_vars/all.yml` sans mettre à jour `platform.yml` dans
  `control-plane`.
- Ne pas committer les fichiers générés dans `packer/output/` ni les fichiers
  d'état Vagrant dans `vagrant/.vagrant/`.
