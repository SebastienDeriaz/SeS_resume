## TPM

### Savoir expliquer le principe des chiffrements symétrique, asymétrique, fonctions de hachage, la signature digitale.

#### symétrique

- Une seul clé pour crypter et décrypter
- Codage par bloc ou par bloc chainé **(figure CBC)** 
- `openssl enc -aes-256-cbc -e -in t.txt -out t.enc` //encrypt
- `openssl enc -aes-256-cbc -d -in t.enc -out t.txt`//decrypt

#### asymétrique

- Deux clés (publique et privée) clé publique disponible par des certificats (CMD pgp)
- Encrypt public -> Decrypt private => confidentialité
- Encrypt private-> Decrypt public => signature digitale

#### hash

Transforme un texte, document en un nombre de N bits unique (SHA-2, SHA-3, Blake2).

`md5sum file => a6a0e8d0522e2c5de921b1c455506320` ou `openssl dgst -md5 file MD5(file)= a6a0e8d0522e2c5de921b1c455506320`  

#### signature

En deux parties: 1. Calcul du HASH puis encryptage avec clé privée. **(figure signature)**

### Connaitre les différentes implémentations des TPM (discrete, integrated, Hypervisor, Software)

- discrete : Circuit dédié 
- integrated : Partie du $\mu C$ qui gère le TPM
- Hypervisor : virtuel fournis par personne fiable
- Software : virtuel pour faire des test pas sécurisé

### Connaitre l’architecture interne d’un TPM

**(figure internal)** 

### Connaitre les différentes hiérarchies des TPM (endorsement, platform, owner, null)

- endorsement : réservé au fabricant du TPM et fixé lors de la fabrication.
- platform : réservé au fabricant de l'hôte et peut être modifier par l'équipementier.
- owner : hiérarchie dédiée à l'utilisateur primaire du TPM peut être modifié en tout temps.
- null : réservé aux clés temporaires (RAM s'efface à chaque redémarrage)

### Savoir créer, utiliser des clés avec un TPM

**(figure createKey)** 

### Connaitre les commandes principales d’un TPM (pas tous les paramètres, mais savoir expliquer ce que font ces commandes, être capable de dessiner ce que font les commandes)

- `tpm2_createprimary -C o -G rsa2048 -c o_prim` // créer un clé primaire owner
- `tpm2_getcap handles-transient` // voir clé dans la RAM
- `tpm2_getcap handles-persistent` // voir clé dans la NV-RAM
- `tpm2_evictcontrol –c o_primary.ctx` // sauver une clé en NV-RAM
- `tpm2_flushcontext` -t // effacer toute la RAM
- `tpm2_create -C o_prim -G rsa2048 -u child_pub –r child_priv` // créer clé enfant
- `tpm2_load -C o_prim -u child_pub -r child_priv -c child` // charger clé enfant
- `shred passwd, rm -f passwd`  #supprimer de l'hôte

### Savoir encrypter-décrypter, signer-vérifier avec un TPM

- `tpm2_rsaencrypt -c child -s rsaes clearfile –o encryptedfile`

- `tpm2_rsadecrypt -c child -s rsaes encryptedfile -o clearfile`

- `tpm2_sign -c child -g sha256 -o file.sign file`

- `tpm2_verifysignature -c child -g sha256 -s file.sign -m file`

### Savoir utiliser les registres PCR

tpm2_pcrreset 0

tpm2_pcrextend 0:sha1=8c83...(hash)

### Savoir sauver des données sur le TPM

tpm2_evictcontrol -c passwd.ctx 0x81010000 -C o  #sauver

tpm2_unseal -c 0x81010000 > passwd  #récuperer

### Savoir sauver des données et les protéger avec une PCR policy

sha1sum passwd à 8c8393ac8939430753d7cb568e2f2237bc62d683

tpm2_pcrreset 0 tpm2_pcrextend 0:sha1=8c8393ac8939430753d7cb568e2f2237bc62d683

tpm2_createprimary -C o –G rsa2048 -c primary

tpm2_startauthsession -S session
tpm2_policypcr -S session -l sha1:0 -L pcr0_policy
tpm2_flushcontext session
tpm2_create -C primary -g sha256 \
-u passwd_pcr0.pub -r passwd_pcr0.priv \
–i passwd -L pcr0_policy
tpm2_load -C primary -u passwd_pcr0.pub \
-r passwd_pcr0.priv -c passwd_pcr
tpm2_evictcontrol -c passwd_pcr0 0x81010000 -C o
tpm2_flushcontext session
shred passwd
rm -f passwd
tpm2_startauthsession --policy-session -S session
tpm2_policypcr -S session -l sha1:0
tpm2_unseal -p session:session -c 0x81010000 > passwd