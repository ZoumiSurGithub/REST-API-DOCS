# REST API

- Bienvenue sur le projet REST API
  d'[MCBE Serv Stats](https://discord.com/api/oauth2/authorize?client_id=865149618936610847&permissions=8&scope=bot%20applications.commands),
  ici vous trouverez toute les informations sur comment metre à jour notre base de donnée pour la liaison de vos plugins
  à discord via **MCBE Serv Stats**.
- Nous vous recommandons d'effectuer les requêtes dans
  une [AsyncTask](https://github.com/pmmp/PocketMine-MP/blob/stable/src/scheduler/AsyncTask.php) pour ne pas pertuber le
  MainThread de [PocketMine-MP](https://github.com/pmmp/PocketMine-MP).

**ATTENTION** : Les requêtes sont à effectuer sur le port 3000.
Pour les exemples ci-dessous nous utilisons notre API pour les requêtes vers la REST API.
---

# Liste des codes

**ATTENTION** : Ce sont les codes que nous envoyons pour signaler si la requête cest passer comme prévue ou pas de notre
coté. Si le serveur ne répond pas alors nous répondons avec un code 500.

- 0: Succès
- 1: Token invalide
- 2: Les données existent déjà
- 3: Joueur non enregistré
- 4: Joueur déjà enregistré

---
# Sommaire
- Plugin Money
  - [Initialisation de la base de données](#initialisation-de-la-base-de-données-pour-le-plugin-money)
  - [Ajouter un joueur](#ajour-dun-joueur)
  - [Récupération du solde d'un joueur](#récupération-du-solde-dun-joueur)
  - [Mettre à jour le solde d'un joueur](#définir-le-solde-dun-joueur)
- Plugin Link
  - [Initialisation de la base de données](#initialisation-de-la-base-de-donnée)
  - [Création du joueur](#création-du-joueur)
  - [Obtenir les informations d'un joueur](#obtenir-les-informations-dun-joueur)

# Plugin Money

---

## Initialisation de la base de données pour le plugin Money

- Requête à effectuer: ``http://moonlight-mcbe.fr:3000/api/money/init/{token}``

```php
    public function onEnable() : void{
        $this->getServer()->getAsyncPool()->submitTask(new PostRequestAsyncTask(
            "http://moonlight-mcbe.fr:3000/api/money/init/" . Money::getToken(),
            ["token" => Money::getToken()],
            function (array $result) {
                if ($result["code"] === 1 || $result["code"] === 404) {
                    $this->getLogger()->error("Token invalid.");
                    $this->getServer()->getPluginManager()->disablePlugin($this);
                } elseif ($result["code"] === 2) {
                    $this->getLogger()->notice("Token correct.");
                } elseif ($result["code"] === 0) {
                    $this->getLogger()->notice("Token correct. The database has been initialized.");
                }
            }
        ));
    }
```

---

## Ajour d'un joueur

- Nous vous recommandons de faire la requête sur l'événement PlayerLoginEvent
- Requête à effectuer: ``http://moonlight-mcbe.fr:3000/api/money/user/create/{token}&{playerName}``

```php
public function onLogin(PlayerLoginEvent $event): void{
    $player = $event->getPlayer();
    Server::getInstance()->getAsyncPool()->submitTask(new PostRequestAsyncTask(
        "http://moonlight-mcbe.fr:3000/api/money/user/create/" . Money::getToken() . "&" . $player->getName(),
        ["token" => Money::getToken(), "username" => $player->getName()]
    ));
}
```

---

## Récupération du solde d'un joueur

- Requête à effectuer: ``http://moonlight-mcbe.fr:3000/api/money/user/get/{token}&{playerName}``

```php
Server::getInstance()->getAsyncPool()->submitTask(new GetRequestAsyncTask(
    "http://moonlight-mcbe.fr:3000/api/money/user/get/" . Money::getToken() . "&" . $sender->getName(),
    ["token" => Money::getToken(), "username" => $sender->getName()],
    function (array $result) use ($sender) {
        if ($result["code"] === 3) {
            $sender->sendMessage("Player not registered.");
        } elseif ($result["code"] === 0) {
            $sender->sendMessage("Your balance is: " . $result["content"]["money"]);
        }
    }
));
```

---

## Définir le solde d'un joueur

- Requête à effectuer: ``http://moonlight-mcbe.fr:3000/api/money/user/set/{token}&{playerName}&{money}``

```php
Server::getInstance()->getAsyncPool()->submitTask(new PostRequestAsyncTask(
    "http://moonlight-mcbe.fr:3000/api/money/user/set/" . Money::getToken() . "&" . $sender->getName() . "&" . $money,
    ["token" => Money::getToken(), "username" => $sender->getName(), "money" => $money]
));
```

---

# Plugin Link

---

## Initialisation de la base de donnée

- Requête à effectuer: ``http://moonlight-mcbe.fr:3000/api/link/init/{token}``

```php
    public function onEnable() : void{
        $this->getServer()->getAsyncPool()->submitTask(new PostRequestAsyncTask(
            "http://moonlight-mcbe.fr:3000/api/link/init/" . Link::getToken(),
            ["token" => Link::getToken()],
            function (array $result) {
                if ($result["code"] === 1 || $result["code"] === 404) {
                    $this->getLogger()->error("Token invalid.");
                    $this->getServer()->getPluginManager()->disablePlugin($this);
                } elseif ($result["code"] === 2) {
                    $this->getLogger()->notice("Token correct.");
                } elseif ($result["code"] === 0) {
                    $this->getLogger()->notice("Token correct. The database has been initialized.");
                }
            }
        ));
    }
```

---

## Création du joueur

- Nous vous recommandons de faire la requête sur l'événement PlayerLoginEvent.
- Requête à effectuer: ``http://moonlight-mcbe.fr:3000/api/link/user/create/{token}&{playerName}``

```php
public function onLogin(PlayerLoginEvent $event): void{
    $player = $event->getPlayer();
    Server::getInstance()->getAsyncPool()->submitTask(new PostRequestAsyncTask(
        "http://moonlight-mcbe.fr:3000/api/link/user/create/" . Link::getToken() . "&" . $player->getName(),
        ["token" => Link::getToken(), "username" => $player->getName()]
    ));
}
```

## Obtenir les informations d'un joueur

- Requête à effectuer: ``http://moonlight-mcbe.fr:3000/api/link/user/get/{token}&{playerName}``

```php
Server::getInstance()->getAsyncPool()->submitTask(new GetRequestAsyncTask(
    "http://moonlight-mcbe.fr:3000/api/link/user/get/" . Link::getToken() . "&" . $sender->getName(),
    ["token" => Link::getToken(), "username" => $sender->getName()],
    function (array $result) use ($sender) {
        if ($result["code"] === 3) {
            $sender->sendMessage("Player not registered.");
        } elseif ($result["code"] === 0) {
            var_dump($result["content"]);
             /** array(3) {
             *   ["username"]=>
             *   string "playerName"
             *   ["discord_id"]=>
             *   string "discordId"
             *   ["booster"]=>
             *   bool
             *   ["code"]=>
             *   string "codeFor/link"
             *   }
             */
        }
    }
));
```