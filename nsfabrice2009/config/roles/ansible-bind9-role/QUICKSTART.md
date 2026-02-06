# Démarrage rapide - Rôle Ansible Bind9

## Installation rapide en 5 minutes

### 1. Préparer l'environnement
```bash
# Extraire l'archive
unzip ansible-bind9-role.zip
cd ansible-bind9-role

# Installer Ansible (si nécessaire)
sudo apt update
sudo apt install ansible
```

### 2. Configuration minimale

Éditez `inventory/hosts.yml` :
```yaml
dns01:
  ansible_host: 192.168.1.10      # ← VOTRE IP ICI
  ansible_user: fabrice           # ← VOTRE USER ICI
```

Éditez `group_vars/dns_servers.yml` :
```yaml
dns_domain: "lab.local"           # ← VOTRE DOMAINE
dns_network: "192.168.1"          # ← VOTRE RÉSEAU

dns_records:
  - name: ns1
    ip: 192.168.1.10              # ← IP de votre DNS
  - name: proxmox
    ip: 192.168.1.20              # ← VOS SERVEURS
  - name: nas
    ip: 192.168.1.30
```

### 3. Déployer
```bash
# Test de connexion
ansible dns_servers -m ping

# Déploiement
ansible-playbook playbook-dns.yml

# Tester
ansible-playbook playbook-dns-test.yml
```

### 4. Configurer vos clients

Sur vos machines, utilisez `192.168.1.10` comme serveur DNS.

**Linux** : 
```bash
echo "nameserver 192.168.1.10" | sudo tee /etc/resolv.conf
```

**Ou configurez votre routeur DHCP** pour distribuer automatiquement le DNS à tous les clients.

### 5. Vérifier
```bash
dig @192.168.1.10 proxmox.lab.local
ping proxmox.lab.local
```

## Commandes utiles

```bash
# Redéployer après modifications
ansible-playbook playbook-dns.yml

# Mode verbeux pour déboguer
ansible-playbook playbook-dns.yml -vv

# Vérification sans appliquer
ansible-playbook playbook-dns.yml --check

# Tester le DNS
dig @192.168.1.10 nas.lab.local
nslookup docker.lab.local 192.168.1.10
```

## Ajouter un serveur

1. Éditez `group_vars/dns_servers.yml`
2. Ajoutez dans `dns_records`:
   ```yaml
   - name: monserveur
     ip: 192.168.1.100
   ```
3. Relancez : `ansible-playbook playbook-dns.yml`

C'est tout ! 🚀
