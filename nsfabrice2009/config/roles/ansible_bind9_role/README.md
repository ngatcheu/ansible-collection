# Rôle Ansible Bind9 pour Homelab

Ce rôle Ansible permet d'automatiser la configuration d'un serveur DNS Bind9 pour votre homelab.

## Fonctionnalités

- Installation automatique de Bind9 et ses utilitaires
- Configuration des zones DNS directe et inverse
- Gestion automatique du numéro de série (timestamp)
- Support des enregistrements A et CNAME
- Configuration des forwarders DNS
- Validation automatique de la configuration

## Prérequis

- Ansible 2.9+
- Système cible : Debian/Ubuntu
- Accès SSH avec privilèges sudo

## Structure du projet

```
ansible-bind9-role/
├── inventory/
│   └── hosts.yml                    # Inventaire des serveurs
├── group_vars/
│   └── dns_servers.yml              # Variables de configuration DNS
├── roles/
│   └── bind9/
│       ├── tasks/
│       │   └── main.yml             # Tâches principales
│       ├── templates/
│       │   ├── named.conf.options.j2
│       │   ├── named.conf.local.j2
│       │   ├── zone-forward.j2
│       │   └── zone-reverse.j2
│       ├── handlers/
│       │   └── main.yml             # Handlers pour redémarrage
│       └── vars/
│           └── main.yml
├── playbook-dns.yml                 # Playbook principal
├── playbook-dns-test.yml            # Playbook de test
└── README.md
```

## Installation

1. **Cloner ou extraire l'archive**
   ```bash
   unzip ansible-bind9-role.zip
   cd ansible-bind9-role
   ```

2. **Configurer l'inventaire**
   
   Éditez `inventory/hosts.yml` et remplacez les valeurs par défaut :
   ```yaml
   dns01:
     ansible_host: 192.168.1.10        # IP de votre serveur DNS
     ansible_user: votre_utilisateur   # Votre utilisateur SSH
   ```

3. **Configurer les variables DNS**
   
   Éditez `group_vars/dns_servers.yml` pour définir :
   - Votre domaine local (`dns_domain`)
   - Votre réseau (`dns_network`)
   - Les forwarders DNS
   - Vos enregistrements DNS

   Exemple :
   ```yaml
   dns_domain: "lab.local"
   dns_network: "192.168.1"
   
   dns_records:
     - name: proxmox
       ip: 192.168.1.20
       type: server
     - name: nas
       ip: 192.168.1.30
       type: server
   ```

## Utilisation

### Déploiement initial

```bash
# Vérifier la syntaxe
ansible-playbook playbook-dns.yml --syntax-check

# Mode dry-run (simulation)
ansible-playbook -i inventory/hosts.yml playbook-dns.yml --check

# Déploiement réel
ansible-playbook -i inventory/hosts.yml playbook-dns.yml

# Avec verbose pour le débogage
ansible-playbook -i inventory/hosts.yml playbook-dns.yml -vv
```

### Ajouter un nouveau serveur

1. Éditez `group_vars/dns_servers.yml`
2. Ajoutez l'enregistrement dans `dns_records` :
   ```yaml
   - name: nouveauserveur
     ip: 192.168.1.50
     type: server
   ```
3. Relancez le playbook :
   ```bash
   ansible-playbook -i inventory/hosts.yml playbook-dns.yml
   ```

Le numéro de série sera automatiquement mis à jour !

### Tester le DNS

Après le déploiement, testez le DNS :

```bash
# Depuis votre poste local
ansible-playbook -i inventory/hosts.yml playbook-dns-test.yml

# Ou manuellement
dig @192.168.1.10 proxmox.lab.local
dig @192.168.1.10 -x 192.168.1.20
```

## Configuration des clients

Une fois le serveur DNS déployé, configurez vos clients pour l'utiliser.

### Linux
Éditez `/etc/resolv.conf` :
```
nameserver 192.168.1.10
```

Pour une configuration permanente, utilisez votre gestionnaire réseau (NetworkManager, systemd-resolved, etc.)

### Windows
Paramètres réseau > Propriétés de la connexion > IPv4 > Serveur DNS préféré : `192.168.1.10`

### macOS
Préférences Système > Réseau > Avancé > DNS > Ajouter `192.168.1.10`

### Via DHCP (recommandé)
Configurez votre routeur/serveur DHCP pour distribuer automatiquement l'adresse de votre DNS à tous les clients.

## Variables disponibles

### Dans group_vars/dns_servers.yml

| Variable | Description | Exemple |
|----------|-------------|---------|
| `dns_domain` | Domaine local | `lab.local` |
| `dns_network` | Réseau IP | `192.168.1` |
| `dns_reverse_zone` | Zone inverse | `1.168.192.in-addr.arpa` |
| `dns_forwarders` | Serveurs DNS externes | `[1.1.1.1, 8.8.8.8]` |
| `dns_allowed_networks` | Réseaux autorisés | `[localhost, 192.168.1.0/24]` |
| `dns_records` | Enregistrements A | Voir exemple ci-dessous |
| `dns_cname_records` | Alias CNAME | Voir exemple ci-dessous |

Exemple d'enregistrement A :
```yaml
dns_records:
  - name: serveur1
    ip: 192.168.1.100
    type: server
```

Exemple d'enregistrement CNAME :
```yaml
dns_cname_records:
  - name: www
    target: serveur1
```

## Dépannage

### Vérifier le statut de Bind9
```bash
ssh votre_utilisateur@192.168.1.10
sudo systemctl status bind9
```

### Consulter les logs
```bash
sudo journalctl -u bind9 -f
```

### Vérifier la configuration
```bash
sudo named-checkconf
sudo named-checkzone lab.local /etc/bind/zones/db.lab.local
```

### Recharger la configuration
```bash
sudo rndc reload
```

## Personnalisation avancée

### Modifier les TTL
Éditez les templates dans `roles/bind9/templates/` pour ajuster les valeurs TTL.

### Ajouter d'autres types d'enregistrements
Modifiez `zone-forward.j2` pour ajouter le support de MX, TXT, SRV, etc.

### Configuration multi-zones
Dupliquez les sections de zone dans `named.conf.local.j2` pour gérer plusieurs domaines.

## Sécurité

- Le rôle configure Bind9 pour n'accepter que les requêtes des réseaux autorisés
- La récursion est activée uniquement pour les réseaux de confiance
- DNSSEC est activé par défaut

## Contribuer

N'hésitez pas à adapter ce rôle à vos besoins spécifiques !

## Licence

Libre d'utilisation pour votre homelab.

## Auteur

Créé avec Ansible pour simplifier la gestion DNS en homelab.
