Cluster = VM k3s-manager sous Vagrant
But = piloter le cluster depuis ton PC


---

🔹 Étape 1 — Quitter la VM si tu es dedans

exit

Tu dois être sur ton PC Ubuntu, là où se trouve le Vagrantfile.


---

🔹 Étape 2 — Récupérer la config SSH de la VM

vagrant ssh-config k3s-manager

Tu verras un bloc comme :

Host k3s-manager
  HostName 127.0.0.1
  User vagrant
  Port 2222
  IdentityFile /home/ubuntu/.../private_key


---

🔹 Étape 3 — Copier le kubeconfig depuis la VM

Toujours sur ton PC Ubuntu :

mkdir -p ~/.kube

scp -P 2222 \
-i /chemin/vers/private_key \
vagrant@127.0.0.1:/etc/rancher/k3s/k3s.yaml \
~/.kube/config

(remplace le port et la clé avec les valeurs de vagrant ssh-config)


---

🔹 Étape 4 — Corriger les droits

chmod 600 ~/.kube/config


---

🔹 Étape 5 — Modifier l’adresse du serveur

nano ~/.kube/config

Remplace :

server: https://127.0.0.1:6443

par l’IP privée de la VM (ex: 192.168.56.10).


---

🔹 Étape 6 — Tester

kubectl get nodes


---

🧠 Résumé

✔ VM = héberge K3s
✔ PC Ubuntu = pilote le cluster
✔ scp sert à sortir le kubeconfig de la VM
✔ Ensuite kubectl fonctionne sans Vagrant


Parfait, on avance bien.
Le kubeconfig est copié. Le nouveau problème est simplement :

zsh: command not found: kubectl

👉 kubectl n’est pas installé sur ton PC hôte
Le cluster K3s a kubectl dans la VM, pas sur ta machine.


---

✅ Installer kubectl sur ton PC Ubuntu

sudo apt update
sudo apt install -y kubectl

Si le paquet n’existe pas :

curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/


---

✅ Vérifier que ton kubeconfig est bien pris en compte

echo $KUBECONFIG

S’il ne renvoie rien → OK (kubectl utilisera ~/.kube/config par défaut)

Sinon :

export KUBECONFIG=~/.kube/config


---

⚠️ IMPORTANT — Adresse du serveur

Ouvre ton kubeconfig :

nano ~/.kube/config

Si tu vois :

server: https://127.0.0.1:6443

➡️ Remplace par l’IP privée de la VM (pas 127.0.0.1), par exemple :

server: https://192.168.56.10:6443

Sinon ton PC parlera à lui-même.


---

🧪 Test final

kubectl get nodes
kubectl get pods -A


---

'''
set -e

ARCH="$(uname -m)"
if [ "$ARCH" != "x86_64" ]; then
  echo "Arch non supportée ici: $ARCH (attendu x86_64/amd64)"
  exit 1
fi

KVER="$(curl -fsSL https://dl.k8s.io/release/stable.txt | tr -d '\r\n')"
echo "Kubectl version: $KVER"

curl -fsSLo kubectl "https://dl.k8s.io/release/${KVER}/bin/linux/amd64/kubectl"
sudo install -m 0755 kubectl /usr/local/bin/kubectl
rm -f kubectl

kubectl version --client
'''
