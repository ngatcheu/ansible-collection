# Rôle Ansible — openldap-ansible-role

Ce rôle installe un serveur **OpenLDAP** (`slapd`) et y charge des données initiales (utilisateurs, groupes, schémas).

> **Usage : développement et tests uniquement** — ne pas utiliser en production.

---

## Prérequis

- Ansible >= 2.3.2
- Module Python `python3-ldap` (installé automatiquement par le rôle)
- Distributions supportées : Debian/Ubuntu, RedHat/CentOS (EL 6/7)

---

## Variables du rôle

### Variables obligatoires

À définir dans le playbook ou l'inventaire :

```yaml
ldap_basedn:      dc=mondomain,dc=net    # Base DN de l'annuaire
ldap_server_uri:  ldap://localhost:389   # URI du serveur LDAP
ldap_bind_pw:     secret                 # Mot de passe administrateur
```

### Variables optionnelles — Utilisateurs et groupes

```yaml
ldap_users:
  jean.dupont:
    cn: Jean Dupont
    givenname: Jean
    sn: Dupont
    mail: jean.dupont@mondomain.net
    userpassword: motdepasse
  marie.martin:
    cn: Marie Martin
    givenname: Marie
    sn: Martin
    mail: marie.martin@mondomain.net
    userpassword: motdepasse

ldap_groups:
  - name: devops
    members:
      - jean.dupont
  - name: admin
    members:
      - jean.dupont
      - marie.martin
```

### Variables internes (par OS)

| Variable        | Debian              | RedHat                                          |
|-----------------|---------------------|-------------------------------------------------|
| `dbtype`        | `{1}mdb`            | `{2}hdb`                                        |
| `schema_path`   | `/etc/ldap/schema`  | `/etc/openldap/schema`                          |
| `dbconfig_path` | `/usr/share/slapd/DB_CONFIG` | `/usr/share/openldap-servers/DB_CONFIG.example` |

---

## Paquets installés

**Debian/Ubuntu :**
- `slapd`
- `ldap-utils`
- `python3-ldap`

**RedHat/CentOS :**
- `openldap`
- `openldap-servers`
- `openldap-clients`
- `python-ldap`

---

## Déroulement des tâches

1. Chargement des variables spécifiques à l'OS
2. Installation des paquets OpenLDAP
3. Démarrage du service `slapd`
4. Chiffrement du mot de passe via `slappasswd`
5. Application du template `db.ldif` (configuration de la base de données)
6. Chargement des schémas standards : `cosine`, `nis`, `inetorgperson`
7. Création de l'entrée racine (DN racine)
8. Création des unités organisationnelles `ou=groups` et `ou=people`
9. Chargement des utilisateurs dans `ou=people`
10. Création des groupes dans `ou=groups`
11. Ajout des membres dans chaque groupe
12. Redémarrage de `slapd`

---

## Structure du répertoire

```
openldap-ansible-role/
├── meta/
│   └── main.yml              # Métadonnées Ansible Galaxy
├── tasks/
│   ├── main.yml              # Orchestration principale
│   ├── Debian.yml            # Installation Debian/Ubuntu
│   └── RedHat.yml            # Installation RedHat/CentOS
├── templates/
│   └── db.ldif               # Template de configuration de la base LDAP
└── vars/
    ├── Debian-vars.yml       # Variables spécifiques Debian
    └── RedHat-vars.yml       # Variables spécifiques RedHat
```

---

## Dépendances

Aucune.

---

## Exemple de playbook

Déclaration dans `requirements.yml` :

```yaml
- src: nsfabrice2009.openldap-ansible-role
  name: openldap-ansible-role
```

Utilisation dans un playbook :

```yaml
- hosts: ldap_servers
  become: true
  vars:
    ldap_basedn: dc=mondomain,dc=net
    ldap_server_uri: ldap://localhost:389
    ldap_bind_pw: "{{ vault_ldap_bind_pw }}"
    ldap_users:
      jean.dupont:
        cn: Jean Dupont
        givenname: Jean
        sn: Dupont
        mail: jean.dupont@mondomain.net
        userpassword: "{{ vault_user_password }}"
    ldap_groups:
      - name: devops
        members:
          - jean.dupont
  roles:
    - openldap-ansible-role
```

---

## Licence

Simplified BSD License

## Auteur

Luis Novo — NSFabrice2009
