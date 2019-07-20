---
layout: post
title:  " Szamlazz.hu Woocommerce Subscription email (Hungarian article)"
author: alienteavend
categories: [ wordpress, tutorial ]
tags: [ Woocommerce, hungarian, szamlazz.hu]
image: assets/images/wordpress.jpg
description: "(hungarian) Számlázz.hu Woocommerce integráció plugin nem küld számlát emailben Woocommerce Subscription rendelés megújításakor"
featured: true
hidden: true
comments: false
---


# Szamlazz.hu Woocommerce Subscription email (Hungarian article)
This is a hungarian article because the plugin being used here is specifically for creating bills in accordance to hungarian tax laws.

---

Hosszú cím, de aki erre keres az remelém megtalálja. Adott a következő probléma: Számlázz.hu Woocommerce integráció plugin nem küld számlát emailben Woocommerce Subscription rendelés megújításakor.

A felállás:
- Woocommerce modul telepítve
-- [Szamlazz.hu integráció](https://szamlazzplugin.hu/) bekötve
-- [Woocommerce Subscriptions Plugin](https://woocommerce.com/products/woocommerce-subscriptions/) bekötve
- Woocommerce Simple Subscription termék beállítva
- Vásárlónál eljött a vásárlás fordulója, kap emailt de nem kap számlát.

Szeretnénk, hogy automatikusan küldjünk e-mailben számlát a felhasználónak nem csak a subscription vásárlásakor, hanem annak megújításakor is. A számlázz.hu jelenlegi verziója viszont ezt nem tudja, legenerálja ugyan a számlát, amit az admin oldalon meg is lehet tekinteni, de nem küldi el a vásárlónak minden automatikus fizetéskor.

A Woocommerce Subscription modell a következő:
![modell](https://docs.woocommerce.com/wp-content/uploads/2013/07/subscription-renewal-process-flow-chart.jpg)

A háttérben a következő történik:
- Woocommerce a megfelelő időpontban létrehoz egy Order-t, amit hozzárendel a felhasználó subscriptionjéhez.
- Az order állapota pending, amíg a fizetés beérkezik.
- A PayPal Standard (vagy egyéb payment gateway) felé megy egy kérés a felhasználó alapértelmezett fizetési lehetőségének terhelésére a megfelelő összeggel.
- Ha ez sikeres, akkor a PayPal küld egy IPN üzenetet a weboldalnak a háttérben.
-- [IPN konfiguráció](https://docs.woocommerce.com/document/paypal-standard/)
- IPN beérkezésekor az validálva van, és az order állapota completed lesz, a subscriptioné pedig újra aktív.
- Számázz integráció készít egy számlát.
- Megy e-mail a vásárlónak, amihez szeretnénk csatolni a számlát.
-- Ez a mail alapból engedélyezve van,  Woocommerce > Settings > Emails > Customer Renewal Invoice

Ha megnyitjuk ezt az emailt `wp-admin/admin.php?page=wc-settings&tab=email&section=wcs_email_completed_renewal_order` linken, itt irja a template nevét amit használ, nevezetesen: `customer_completed_renewal_order` ezt kell behorgászni a számlázz integrációba, kebab-case helyett lower_case.

A következőt kell módosítani:
`wp-plugins/integration-for-szamlazzhu-woocommerce/index.php`
@ 2382 sor:

```
		if( $email_id === 'customer_completed_order' || $email_id === 'customer_invoice'){
```
Erre kell átírni:
```
		if( $email_id === 'customer_completed_order' || $email_id === 'customer_invoice' || $email_id == 'customer_completed_renewal_order'){
```

Voilá. A legközelebbi emaillel számla is megy. Érdemes kiprobálni 1 napos subscriptionnel, $0.01 helyett (mert ez egy speciális érték) mondjuk $0.02 termék árral.

