+++
url = "/fr/plate-forme/avancé/sites/utiliser-le-pare-feu-applicatif_waf/"
title = "Comment utiliser le Pare-feu applicatif web (WAF)"
menuTitle = "Utiliser le WAF"
layout = "howto"
weight = 10
draft = false
tags = ["http", "site", "waf"] 
+++

Pour paramétrer le pare-feu applicatif web, cela se passe sur l'interface d'administration dans **Web > Sites > Détails du site > WAF**. 

## Profils disponibles

|Profil|Description|
|----|----|
|Aucun|(par défaut)|
|Basique|Respect strict du pro­to­cole|
||HTTP Détection de robots mal­veillants|
|Fort|L’ensemble des règles du pro­fil basique|
||Détection d’exécution de code à dis­tance (RCE)|
||Détection d’attaque type [Cross-Site Scripting (XSS)](https://fr.wikipedia.org/wiki/Cross-site_scripting)|
||Détection d’[injec­tion SQL](https://fr.wikipedia.org/wiki/Injection_SQL)|
|Complet|L’ensemble des règles du pro­fil fort|
||Détection d’attaques rela­tives au lan­gage PHP|
||Détection d’attaque par inclu­sion de fichier local (LFI)|
||Détection d’attaque par [inclu­sion de fichier dis­tant (RFI)](https://fr.wikipedia.org/wiki/Remote_File_Inclusion)|
|WordPress|L’ensemble des règles du pro­fil com­plet|
||Règles spé­ci­fiques à WordPress|
|Drupal|L’ensemble des règles du pro­fil com­plet|
||Règles spé­ci­fiques à Drupal|

## Exclure des règles

Selon votre cas d'utilisation, le **comportement du WAF peut être trop restrictif**. Il est aussi possible qu'il génère de **faux positifs** lors de son analyse. Si vous jugez que son comportement n'est pas approprié, vous avez la possibilité d'exclure certaines règles utilisées lors de l'analyse.

Seul le **numéro de la règle à exclure** doit être spécifié. Vous le retrouverez dans les logs Sites (_~/admin/logs/sites_). Exemple :

<font size="-1">

* [08/Jan/2019:11:09:19 +0100] [waf] - \<IP attaquante\> "GET /?param=%22>\<script\>alert(1);\</script\> HTTP/1.1" - 941100 | XSS Attack Detected via libinjection' with value: ">\<script\>alert(1);\</script>
* [08/Jan/2019:11:09:19 +0100] [waf] - \<IP attaquante\> "GET /?param=%22>\<script\>alert(1);\</script\> HTTP/1.1" - 941110 | XSS Filter - Category 1: Script Tag Vector' with value: \<script\>
* [08/Jan/2019:11:09:19 +0100] [waf] - \<IP attaquante\> "GET /?param=%22>\<script\>alert(1);\</script\> HTTP/1.1" - 941160 | NoScript XSS InjectionChecker: HTML Injection' with value: \<script\>

</font>

Ce serait donc 941100, 941110 et 941160 qui pourraient être indiqués.

> IMPORTANT : il faut veiller à ajouter progressivement des règles car l'exclusion est applicable sur tout le site. En effet, même si ajouter un grand nombre de règles à exclure peut améliorer la navigation dans certains cas, la protection sera alors amoindrie dans tous les autres cas.


## Exclure des chemins

Ce type d'exclusion permet d'**éviter l'analyse de pages commençant par le chemin spécifié**. En saisissant <font color=red>/foo/</font> par exemple, <font color=grey>_www.mon-site.com/foo/_</font> sera exclu de l'analyse tout comme les query strings : <font color=grey>_www.mon-site.com/foo/?param=bar_</font>. Pour exclure aussi <font color=grey>-www.mon-site.com/foo/bar_</font> et <font color=grey>_www.mon-site.com/foo/script.php_</font>, il faut rajouter un wildcard : <font color=red>/foo/\*</font>. Enfin, si on veut substituer un caractère quelconque (notamment qui changerait régulièrement), <font color=red>?</font> peut être utilisé.<br>
Donc, pour écarter de l'analyse <font color=grey>_www.mon-site.com/whatever/fooT/_</font>, _whatever_ et _T_ étant des strings quelconques, le chemin à exclure serait : <font color=red>/\*/foo?/</font>.

Exemple : prenons le cas d'un site de type WordPress qui présente des logs similaires à ceux présentés précédemment. Si ces règles sont déclenchées lors de la navigation dans l'interface d'administration du blog, alors il est possible de les exclure de manière permanente.<br>
Cependant, le blog en lui-même ne sera plus protégé contre ces tentatives d'attaques. Dans ce cas, il est plus judicieux d'exclure le chemin (exemple : /wp-admin/\*) pour que toutes vos opérations sur l'interface d'administration ne soient plus concernées par l'analyse du WAF.


## Exclure des IP

Il peut être intéressant d'exclure des **IP sûres** pour éviter à des outils ou des personnes d'être bloqués. 

Prenons l'exemple de [WPScan](https://wpscan.org/) : en l'activant sur un site WordPress certaines des requêtes qu'il effectue peuvent être bloquées. Exclure des règles ou des chemins ne serait pas efficace comme il observe de nombreuses URLs. La solution est donc d'exclure le serveur HTTP sur lequel est installé WPScan pour qu'il puisse fonctionner normalement.