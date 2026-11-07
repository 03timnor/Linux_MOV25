# Linux, användare, diskar och databas

Repository: https://github.com/03timnor/Linux_MOV25

*Tim Noreliusson Lingestedt, 2026-11-07*

Linux-administration på en *EC2*-server (*Amazon Linux*).

## Modul 1: Användare, behörigheter och åtkomst

### *__1. Kommandon__*

#### Skapa nyckelpar:

*__OBS!__* På ens egna dator via en separat *bash* terminal.

```bash
cd ~/.ssh
ssh-keygen -t ed25519 -f ops-anna
cat ops-anna.pub
```

#### Ta fram servrarnas IP-adresser:

```bash
aws ec2 describe-instances --region eu-north-1 \
  --filters Name=tag:Name,Values=novatrix-bastion,novatrix-webb Name=instance-state-name,Values=running \
  --query 'Reservations[].Instances[].[Tags[?Key==`Name`].Value|[0],PublicIpAddress,PrivateIpAddress]' \
  --output table
```

#### Skapa *ops-anna* på *Bastion*:

- Logga in på *Bastion* (*ec2-user*):
```bash
ssh -i ~/.ssh/AWS_MOV25.pem ec2-user@51.20.82.182
```

- Skapa konto, grupp, lägg till publik *SSH* nyckel på servern och ge *ops-anna* rättigheter till nycklen:
```bash
sudo groupadd drift
sudo useradd -m -G drift ops-anna
sudo mkdir -p /home/ops-anna/.ssh
echo "ssh-ed25519 FYLL I PUBLIK SSH NYCKEL" | sudo tee /home/ops-anna/.ssh/authorized_keys
sudo chown -R ops-anna:ops-anna /home/ops-anna/.ssh
sudo chmod 700 /home/ops-anna/.ssh
sudo chmod 600 /home/ops-anna/.ssh/authorized_keys
exit
```

#### Skapa *ops-anna* på webbserver:

- Logga in på *Webbserver* (*ec2-user*):
```bash
ssh -i ~/.ssh/AWS_MOV25.pem \
  -o ProxyCommand="ssh -i ~/.ssh/AWS_MOV25.pem -W %h:%p ec2-user@51.20.82.182" \
  ec2-user@10.0.2.50
```

- Skapa konto, grupp, lägg till publik *SSH* nyckel på servern och ge *ops-anna* rättigheter till nycklen:

```bash
findmnt /var/www/html
sudo groupadd drift
sudo useradd -m -G drift ops-anna
id ops-anna
sudo mkdir -p /home/ops-anna/.ssh
echo "ssh-ed25519 FYLL I PUBLIK SSH NYCKEL" | sudo tee /home/ops-anna/.ssh/authorized_keys
sudo chown -R ops-anna:ops-anna /home/ops-anna/.ssh
sudo chmod 700 /home/ops-anna/.ssh
sudo chmod 600 /home/ops-anna/.ssh/authorized_keys
sudo cat /home/ops-anna/.ssh/authorized_keys
```

#### Skapa *SSH*-config:

*__OBS!__* På ens egna dator via en separat *bash* terminal.

```bash
cat > ~/.ssh/config <<'EOF'
Host novatrix-bastion
    HostName 51.20.82.182
    User ops-anna
    IdentityFile ~/.ssh/ops-anna

Host novatrix-webb
    HostName 10.0.2.50
    User ops-anna
    IdentityFile ~/.ssh/ops-anna
    ProxyJump novatrix-bastion
EOF
cat ~/.ssh/config
```

#### Logga in som *ops-anna*:

```bash
ssh novatrix-webb
```

#### Skapa *sudo*-regel:

```bash
sudo visudo -f /etc/sudoers.d/drift  # innehåll: %drift ALL=(ALL) NOPASSWD: ALL
```

#### *sudo*-kontroll:

```bash
sudo chmod 0440 /etc/sudoers.d/drift
sudo whoami
sudo visudo -c
sudo -l -U ops-anna
```

#### *SSH*-inställningar:

```bash
sudo sshd -T | grep -iE "authentication|forwarding|permitroot"
sudo cat /etc/ssh/sshd_config.d/10-novatrix.conf
```

#### Lösenordsinloggning-kontroll:

```bash
ssh -o PubkeyAuthentication=no -o PreferredAuthentications=password novatrix-webb
ssh -o PubkeyAuthentication=no -o PreferredAuthentications=password ops-anna@51.20.82.182
```

#### /var/www/html-kontroll:

```bash
findmnt /var/www/html
ls -la /var/www/html
stat -c '%U:%G %a %n' /var/www/html/wp-config.php
sudo -u ops-anna grep DB_PASSWORD /var/www/html/wp-config.php | sed -E "s/'[^']{4,}'/'********'/"
sudo find /var/www/html -perm -o+w
```

#### *chmod* och *chown* test:

```bash
sudo chmod 600 /var/www/html/wp-config.php
stat -c '%U:%G %a %n' /var/www/html/wp-config.php
sudo -u ops-anna grep -c DB_PASSWORD /var/www/html/wp-config.php
sudo chown root /var/www/html/wp-config.php
sudo chmod 640 /var/www/html/wp-config.php
stat -c '%U:%G %a %n' /var/www/html/wp-config.php
```

#### Uppdaterings-kontroll:

```bash
cat /etc/os-release
rpm -q system-release
dnf check-release-update
```

## Modul 2: En extra disk



## Modul 3: Databasen från kommandoraden

