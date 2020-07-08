# 2FA na przykładzie Google Auth Provider

## Wykorzystane oprogramowanie (testowane wersje):
1. NGiNX (1.16.1 - pobrana z repozytorium CentOS)
2. oauth2_proxy (https://github.com/pusher/oauth2_proxy/releases - 4.0.0)
## Konfiguracja NGiNX
1. Do katalogu `/etc/nginx/conf.d/` przekopiować ng_oauth2_proxy.conf
2. Ustawić `ssl_certificate` i `ssl_certificate_key`.
W momencie kiedy SSL ustawiony jest przy pomocy nginx proxy Kibanę można uruchoć z http. Jeśli jednak ma być mimo to uruchomiona z szyfrowaniem trzeba zmienić również `proxy_pass` na odpowiednie.
## Konfiguracja oauth2_proxy
1. Stworzyć katalog, w którym będzie umieszczony program i jego konfiguracja:
```bash
mkdir -p /usr/share/oauth2_proxy/
mkdir -p /etc/oauth2_proxy/
```
2. Przekopiować pliki do katalogów:
```bash
cp oauth2_proxy /usr/share/oauth2_proxy/
cp oauth2_proxy.cfg /etc/oauth2_proxy/
```
3. Ustawić dyrektywy w zgodznie z konfiguracją OAuth w projekcie Google Cloud
```cfg
client_id =
client_secret =
# poniższe ogranicza domeny do autoryzacji (* - wszystkie)
email_domains = [
  "*"
]
# poniższe należy ustawić zgodnie z publiczną nazwą hosta:
cookie_domain = "logserver-kibana-host.org"
```
4. W przypadku jeśli chcemy by istaniały ograniczenia logowania dla konkretnej grupy zdefiniowanej po stronie Google:
- Stworzyć konto serwisowe: https://developers.google.com/identity/protocols/OAuth2ServiceAccount ; pobrać konfigurację konta w pliku json, skopiować Client ID
- Na stronie dashboardu Google Cloud wybrać "APIs & Auth" -> "APIs"
- Kliknąć "Admin SDK" i "Enable API"
- Wykonać instrukcję https://developers.google.com/admin-sdk/directory/v1/guides/delegation#delegate_domain-wide_authority_to_your_service_account i nadać kontu serwisowemu poniższe uprawnienia:
```
https://www.googleapis.com/auth/admin.directory.group.readonly
https://www.googleapis.com/auth/admin.directory.user.readonly
```
- Wykonać instrukcję by nadać dostęp do Admin API https://support.google.com/a/answer/60757
- Stworzyć lub wybrać istniejący email administracyjny w domenie Gmail by nadać mu flagę `google-admin-email`
- Stworzyć lub wybrać istniejącą grupę by nadać jej flagę `google-group`
- Przekopiować pobrany wcześniej plik JSON do `/etc/oauth2_proxy/`. W pliku `oauth2_proxy.cfg` wpisać odpowiednią ścieżkę w:
```
google_service_account_json =
```
## Uruchomienie
1. Uruchomić serwis NGiNX
2. Uruchomić oauth2_proxy
```bash
/usr/share/oauth2_proxy/oauth2_proxy -config="/etc/oauth2_proxy/oauth2_proxy.cfg"
```

3. W przeglądarce wpisać adres kierujący do serwera z instalacją CyberX
