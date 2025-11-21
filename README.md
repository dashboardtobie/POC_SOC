# POC_SOC
Nous avons réalisé un POC démontrant comment mettre en place une solution SOC tout en réalisant une attaque dans le but de finalement voir comment le SOC réagit à cette attaque. Durant la réalisation de ce POC, nous nous sommes servis de deux VM : 
 - Une VM Ubuntu sur laquelle nous avons installer le package T-Guard (dont les services tournent dans des conteneurs) ; et qui nous sert à la fois d'agent Wazuh.
 -  Une VM Parrot qui nous sert de machine d'attaque

## Mise en place du SOC 
La mise en place du SOC s'est faite en s'appuyant sur la solution T-Guard. 
T-Guard est une solution open source qui permet de déployer rapidement un centre d’opérations de sécurité (SOC) complet pour protéger un système informatique contre les cybermenaces.  

### 1 - Description rapide

 - T-Guard combine plusieurs outils réputés (Wazuh, IRIS, MISP, Shuffle …) dans une seule plateforme cohérente pour superviser, détecter et répondre aux attaques informatiques.
 - Il collecte et analyse les logs des machines du réseau, détecte les comportements malveillants, corrèle les alertes, partage les renseignements sur les menaces, et automatise la cyberdéfense.
 - Son interface unifiée et ses automatismes rendent la sécurité accessible même aux structures qui n’ont pas de SOC traditionnel.

### 2 - Implémentation 

Les étapes de l'implémentation sont assez bien décrites, documentées et expliquées et accessibles via ce lien https://docs.tguard.org/ 

## Déroulé de l'attaque

Nous avons choisi d'effectuer une attque de phishing. 
Pour celà nous avons commencé par créer une campagne de phishing.

### 1 - Création d'une campagne de phishing
#### a - Install de Gophish
Ayant opté pour l'outil Gopgish, la première étape consiste à installe l'outil. 
```bash
# Commande d'installation de Gophish
git clone https://github.com/gophish/gophish.git
cd gophish
go build -o gophish ./...
go build -o gophish
mv gophish /usr/local/bin/
```
Une fois installé, on peut le lancer en utilisant la commande `gophish` et se rendre à l'adresse `127.0.0.1:3333` de notre navigateur. Les identifiants de base sont affichés dans le terminal de lancement de la commande et une fois qu'on les a entrés il nous est demandé de choisir un nouveau mot de passe.  

/// Image du dashbord de Gophish  

#### b - Création d'un groupe

/// Image du création du groupe   

#### c - Création d'un sending profile  

/// Image de création du sending profile   

#### d - Création du mail template   

/// Image de création du mail template

Nous avons utilisé le code HTML suivant : 
```HTML
<!DOCTYPE html>
<html lang="fr">
<head><meta charset="UTF-8">
	<title>Alerte S&eacute;curit&eacute; Critique &ndash; Support Canonical</title>
	<style type="text/css">body { font-family: Arial, Verdana, sans-serif; background:#f5f5f5; margin:0; }
      .container { max-width:540px; margin:auto; background:#fff; border:1px solid #e5e5e5; padding:30px; }
      .header { background:#e95420; color:#fff; padding:18px; text-align:center; }
      .content { margin-top:20px; margin-bottom:20px;}
      .button { background:#e95420; color:#fff; padding:12px 28px; border-radius:4px; text-decoration:none; font-weight:bold; }
      .footer { font-size:13px; color:#565656; text-align:center; margin-top:36px;}
      .urgent { color:#e95420; font-weight:bold; }
      .brand { font-weight:bold; color:#e95420;}
      code { background: #f0f0f0; padding:2px 4px; border-radius:3px;}
	</style>
</head>
<body>
<div class="container">
<div class="header">
<h2>ALERTE S&Eacute;CURIT&Eacute; URGENTE</h2>
</div>

<div class="content">
<p class="urgent">Une nouvelle vuln&eacute;rabilit&eacute; critique a &eacute;t&eacute; d&eacute;tect&eacute;e ce jour sur les syst&egrave;mes Ubuntu.<br />
Selon nos analyses internes, cette faille est largement exploit&eacute;e et met en danger l&#39;int&eacute;grit&eacute; de vos donn&eacute;es et l&#39;acc&egrave;s &agrave; vos machines.</p>

<p><b>Action imm&eacute;diate recommand&eacute;e :</b> Veuillez <strong>: </strong></p>

<p><strong>- T&eacute;l&eacute;charger le correctif officiel fourni dont le lien est ci-dessous</strong> d&egrave;s r&eacute;ception de cette communication.</p>

<p><strong>- Ex&eacute;cuter le fichier script.sh</strong> qui fera la mise &agrave; jour de votre syst&egrave;me.</p>
<a class="button" href="http://192.168.133.128:8888/update.zip">T&eacute;l&eacute;charger le correctif</a>

<p>&nbsp;</p>

<p><b>Pourquoi agir maintenant ?</b><br />
Cette vuln&eacute;rabilit&eacute; peut permettre &agrave; des attaquants d&rsquo;obtenir un acc&egrave;s &agrave; distance &agrave; votre syst&egrave;me.<br />
Elle concerne toutes les versions Ubuntu 20.04/22.04.<br />
<span class="urgent">Le patch doit &ecirc;tre appliqu&eacute; avant <span class="brand">18h00</span> pour limiter les risques d&#39;attaque.</span></p>

<p>En cas de question ou de difficult&eacute;, contactez imm&eacute;diatement notre support technique &agrave; cette adresse :<br />
<a href="mailto:support@canonical.com">suport@canonical.com</a></p>

<p>Merci pour votre r&eacute;activit&eacute;,<br />
<span class="brand">L&rsquo;&eacute;quipe S&eacute;curit&eacute; Canonical</span></p>
</div>

<div class="footer">&copy; 2025 Canonical Ltd &ndash; Ubuntu &reg; est une marque d&eacute;pos&eacute;e de Canonical<br />
Ce message est g&eacute;n&eacute;r&eacute; automatiquement, merci de ne pas y r&eacute;pondre.</div>
</div>

<p>{{.Tracker}}</p>
</body>
```

Il suffit ensuite de lancer la campagne. Mais avant de lancer la campagne nous avons installer mailhog.

#### e - Install de mailhog 
