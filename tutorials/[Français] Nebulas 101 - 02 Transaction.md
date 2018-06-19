# Nebulas 101 - 02 Envoyer des Transactions sur Nebulas

[Youtube Tutorial](https://www.youtube.com/watch?v=-44tVVR6ETo&list=PLFipfN18ZQwsW1_dge4w7dfsVNdNZZ37R&index=1)

> Pour suivre ce tutoriel, veuillez a bien avoir respecté l'installation de la première partie [Installation](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2001%20Installation.md).

Nebulas fournit trois méthodes pour envoyer des transactions:

1. Signer & Envoyer
2. Envoyer avec une Passphrase
3. Dévérouiller & Envoyer

Ce tutoriel constitue une introduction pour vous permettre d'envoyer une transaction sur Nebulas en utilisant les trois méthodes listées ci-dessus tout en vérifiant que la transaction est réussie.

## Préparer vos comptes

Sur Nebulas, chaque adresse représente un compte unique.

Préparez deux comptes: une adresse pour envoyer des tokens (l'adresse d'envoi, appelé "from") et l'adresse pour recevoir des tokens (l'adresse de reception, appelée "to").

### L'emetteur

Ici nous allons utiliser le compte coinbase paramètré dans `conf/example/miner.conf` qui défini `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE` en tant qu'emetteur. En tant que "mineur", le compte recevra des tokens en récompense. Ensuite vous pourrez envoyer ces tokens à un autre compte plus tard.

### Le récepteur

Creez un nouveau portefeuille pour recevoir les tokens.

```bash
$ ./neb account new
Your new account is locked with a passphrase. Please give a passphrase. Do not forget this passphrase.
Passphrase:
Repeat passphrase:
Address: n1SQe5d1NKHYFMKtJ5sNHPsSPVavGzW71Wy
```

> Quand vous lancez cette commande, vous allez obtenir un nouveau portefeuille avec l'adresse suivante: `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE`. Nous allons utiliser cette adresse en tant que récepteur.

Votre clef privée "keystore file" de votre nouveau portefeuille se situe dans: `$GOPATH/src/github.com/nebulasio/go-nebulas/keydir/`

## Démarrez les noeuds

### Démarrez le noeud "Seed"

Premièrement, démarrez le noeud "seed" en tant que noeud principal.

```bash
./neb -c conf/default/config.conf
```

### Démarrez le noeud "Miner"

Deuxièmement, démarrez un noeud de minage connecté au noeud "seed". Ce noeud générera de nouveaux blocs sur votre chaine locale.

```bash
./neb -c conf/example/miner.conf
``` 

> **Combien de temps doit-on attendre jusqu'à ce qu'un nouveau bloc soit miné?**
> 
> Nebulas a choisi comme consensus temporaire le DPoS avant que le consensus Proof-of-Devotion soit prêt (PoD, described in [Technical White Paper](https://nebulas.io/docs/NebulasTechnicalWhitepaper.pdf)). Avec cet algorithme de consensus, chaque mineurs pourront miner un nouveau bloc toutes les 15 secondes.
> 
> Dans ce contexte, nous devons attendre 315(=15*21) secondes pour obtenir un nouveau bloc parce qu'il a seulement un mineur en activité parmis les 21 mineurs définis dans `conf/default/genesis.conf`.

Une fois qu'un nouveau bloc a été miné par le mineur, la récompense de minage sera ajoutée au portefeuille de l'adresse "coinbase" défini dans `conf/example/miner.conf`. Dans notre cas il s'agit de `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE`.

## Interagir avec les noeuds

Nebulas fournit aux développeurs des api HTTP et gRPC ainsi qu'une CLI pour interagir avec les noeuds en activité. Nous allons détaillé ici trois méthodes avec les api HTTP ([API Module](https://github.com/nebulasio/wiki/blob/master/rpc.md) | [Admin Module](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md)). 

> Le "listener" HTTP est défini dans la configuration du noeud. Le port par défaut de votre noeud "seed" est `8685`.

Avant tout, vérifiez la balance du compte émetteur avant d'envoyer une transaction.

### Vérifier l'état d'un compte

Récupérez l'état du compte émetteur `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE` en utilisant une rquête `curl` sur l'url suivante: `/v1/user/accountstate`.

```bash
> curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE"}'

{
    "result": {
        "balance": "67066180000000000000",
        "nonce": "0",
        "type": 87
    }
}
```

> **Note**
> Le paramètre "type" est utilisé pour vérifier si le compte est un compte "smart contract". `88` représente un compte "smart contract", et `87` un compte qui n'est pas un "smart contract".

Comme vous pouvez le voir, l'émetteur a été recompensé avec plusieurs tokens pour le minage de nouveaux blocs.

Vérifions maintenant l'état du compte récépteur.

```bash
> curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"your_address"}'


{
    "result": {
        "balance": "0",
        "nonce": "0",
        "type": 87
    }
}
```

Le nouveau compte n'a aucun token.

### Envoyer une Transaction

Maintenant, essayons de transférer les tokens de l'émetteur au récepteur avec les trois méthodes.

#### Signer & Envoyer

Avec cette méthode, il est possible de signer une transaction même avec un environement déconnecté et l'envoyer via un autre noeud en ligne. C'est la méthode la plus sûre pour tout ceux qui souhaitent soumettre une transaction sans exposer votre clef privée sur Internet.

Premièrement, signez la transaction pour obtenir une donnée brute.

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/sign -d '{"transaction":{"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "value":"1000000000000000000","nonce":1,"gasPrice":"1000000","gasLimit":"2000000"}, "passphrase":"passphrase"}'

{"result":{"data":"CiAbjMP5dyVsTWILfXL1MbwZ8Q6xOgX/JKinks1dpToSdxIaGVcH+WT/SVMkY18ix7SG4F1+Z8evXJoA35caGhlXbip8PupTNxwV4SRM87r798jXWADXpWngIhAAAAAAAAAAAA3gtrOnZAAAKAEwuKuC1wU6CAoGYmluYXJ5QGRKEAAAAAAAAAAAAAAAAAAPQkBSEAAAAAAAAAAAAAAAAAAehIBYAWJBVVuRHWSNY1e3bigbVKd9i6ci4f1LruDC7AUtXDLirHlsmTDZXqjSMGLio1ziTmxYJiLj+Jht5RoZxFKqFncOIQA="}}
```

> **Note**
> `None` est un paramètre trés important pour une transaction. il est conçu pour prévenir des [attaques relais](https://en.wikipedia.org/wiki/Replay_attack). For a given account, only after its transaction with nonce N has been accepted, will its transaction with nonce N+1 be processed. Thus, we have to check the latest nonce of the account on chain before preparing a new transaction.

Then, send the raw data to an online Nebulas node.

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/user/rawtransaction -d '{"data":"CiAbjMP5dyVsTWILfXL1MbwZ8Q6xOgX/JKinks1dpToSdxIaGVcH+WT/SVMkY18ix7SG4F1+Z8evXJoA35caGhlXbip8PupTNxwV4SRM87r798jXWADXpWngIhAAAAAAAAAAAA3gtrOnZAAAKAEwuKuC1wU6CAoGYmluYXJ5QGRKEAAAAAAAAAAAAAAAAAAPQkBSEAAAAAAAAAAAAAAAAAAehIBYAWJBVVuRHWSNY1e3bigbVKd9i6ci4f1LruDC7AUtXDLirHlsmTDZXqjSMGLio1ziTmxYJiLj+Jht5RoZxFKqFncOIQA="}'

{"result":{"txhash":"1b8cc3f977256c4d620b7d72f531bc19f10eb13a05ff24a8a792cd5da53a1277","contract_address":""}}
```

#### Send with Passphrase

Si vous accepter de faire confiance au noeud Nebulas, au point de lui transmettre votre clef privée, la seconde méthode sera la plus adaptée.

Premièrement, publiez votre clef privée dans les répertoires "keydir" dans du noeud Nebulas de confiance.

Ensuite, envoyez la transaction avec votre passphrase.


```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/transactionWithPassphrase -d '{"transaction":{"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "value":"1000000000000000000","nonce":2,"gasPrice":"1000000","gasLimit":"2000000"},"passphrase":"passphrase"}'

{"result":{"txhash":"3cdd38a66c8f399e2f28134e0eb556b292e19d48439f6afde384ca9b60c27010","contract_address":""}}
```

> **Note**
> Du fait d'avoir envoyé une transaction avec un paramètre `none` à 1 du compte `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE`, la nouvelle transaction avec le même paramètre `from` doit être incrémentée de 1 (et donc doit avoir comme valeur 2).

#### Dévérouiller & Envoyer

C'est la méthode la plus dangereuse. Vous ne devriez surement pas l'utiliser si nous n'avons totalement confiance dans le noeud Nebulas.

Premièrement, envoyer votre clef privée dans les répertoires keydir du noeud Nebulas.

Ensuite dé-vérouillez vos comptes avec votre passphrase pour une durée donnée.
L'unité de durée est la nano-seconde (300000000000=300s)

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/account/unlock -d '{"address":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","passphrase":"passphrase","duration":"300000000000"}'

{"result":{"result":true}}
```

Une fois le compte dévérouillé, n'importe qui est capable d'envoyer une transaction directement à ce noeud dans la plage de temps indiqué sans autorisation. 

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/transaction -d '{"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "value":"1000000000000000000","nonce":3,"gasPrice":"1000000","gasLimit":"2000000"}'

{"result":{"txhash":"8d69dea784f0edfb2ee678c464d99e155bca04b3d7e6cdba6c5c189f731110cf","contract_address":""}}
```

## Réception d'une transaction

Nous allons récupérer un paramètre `txhash` dans les trois méthodes indiquées ci-dessus aprés que la transaction soit correctement envoyée. Le paramètre `txhash` peut être utilisé pour requêter et obtenir le statut de la transaction.

```bash
> curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/getTransactionReceipt -d '{"hash":"8d69dea784f0edfb2ee678c464d99e155bca04b3d7e6cdba6c5c189f731110cf"}'

{"result":{"hash":"8d69dea784f0edfb2ee678c464d99e155bca04b3d7e6cdba6c5c189f731110cf","chainId":100,"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5","value":"1000000000000000000","nonce":"3","timestamp":"1524667888","type":"binary","data":null,"gas_price":"1000000","gas_limit":"2000000","contract_address":"","status":1,"gas_used":"20000"}}
```

Le champs `status` peut être égal à 0, 1 ou 2.

- **0: Failed.** Indique que la transaction a été envoyée sur la chaine mais que son execution a échoué.
- **1: Successful.** Indique que la transaction a été envoyée sur la chaine et que son execution a réussi.
- **2: Pending.** Indique que la transaction n'a pas été répliquée sur un bloc.

### Double Check

Vérifions encore une fois la balance du récépteur.

```bash
> curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5"}'

{"result":{"balance":"3000000000000000000","nonce":"0","type":87}}
```

Ici vous devez voir que la balance correspond à la somme de toutes les transactions executées auparavant.

### Prochaine étape: Tutorial 3

 [Ecrire et executer un smart contact avec JavaScript](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2003%20Smart%20Contracts%20JavaScript.md)
