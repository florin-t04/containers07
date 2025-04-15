# Lucrare de laborator: Aplicație PHP multi-container cu Docker Compose

## Numele lucrării
Aplicație PHP multi-container cu Docker Compose

## Scopul lucrării
Familiarizarea cu gestionarea unei aplicații multi-container folosind Docker Compose, prin configurarea și rularea unei aplicații PHP împreună cu server web (nginx) și baza de date (MariaDB/MySQL).

## Sarcina
Creați o aplicație PHP bazată pe trei containere:  
- **nginx** (frontend)  
- **php-fpm** (backend)  
- **mariadb/mysql** (bază de date)  

Folosind Docker Compose, se vor configura volumele, rețelele și variabilele de mediu necesare pentru funcționarea aplicației.

## Descrierea efectuării lucrării

1. **Pregătire**
   - Se creează un nou repository numit `containers06` și se clonează pe computer.
   - Asigurați-vă că Docker este instalat și funcțional pe sistem.

2. **Configurarea site-ului PHP**
   - În directorul `containers06`, se creează directorul `mounts/site`.
   - Se copiază codul site-ului PHP (reprezentativ pentru disciplina PHP) în `mounts/site`.

3. **Setarea fișierelor de configurare**
   - Se creează fișierul `.gitignore` în rădăcina proiectului și se adaugă:
     ```
     # Ignore files and directories
     mounts/site/*
     ```
   - Se creează fișierul `nginx/default.conf` în directorul `containers06/nginx` cu următorul conținut:
     ```nginx
     server {
         listen 80;
         server_name _;
         root /var/www/html;
         index index.php;
         location / {
             try_files $uri $uri/ /index.php?$args;
         }
         location ~ \.php$ {
             fastcgi_pass backend:9000;
             fastcgi_index index.php;
             fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
             include fastcgi_params;
         }
     }
     ```
   - Se creează fișierul `docker-compose.yml` în directorul `containers06` cu următorul conținut:
     ```yaml
     version: '3.9'

     services:
       frontend:
         image: nginx:1.19
         volumes:
           - ./mounts/site:/var/www/html
           - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
         ports:
           - "80:80"
         networks:
           - internal
         # Se va adăuga referința către app.env mai jos (exemplu):
         env_file:
           - app.env

       backend:
         image: php:7.4-fpm
         volumes:
           - ./mounts/site:/var/www/html
         networks:
           - internal
         env_file:
           - mysql.env
           - app.env

       database:
         image: mysql:8.0
         env_file:
           - mysql.env
         networks:
           - internal
         volumes:
           - db_data:/var/lib/mysql

     networks:
       internal: {}

     volumes:
       db_data: {}
     ```
   - Se creează fișierul `mysql.env` în rădăcina proiectului și se adaugă următoarele linii:
     ```
     MYSQL_ROOT_PASSWORD=secret
     MYSQL_DATABASE=app
     MYSQL_USER=user
     MYSQL_PASSWORD=secret
     ```

4. **Adăugarea fișierului app.env**
   - Pentru a seta variabila de mediu **APP_VERSION** pentru serviciile **backend** și **frontend**, se creează în rădăcina directorului `containers06` un fișier numit `app.env` cu conținutul:
     ```
     APP_VERSION=1.0.0
     ```
   - În fișierul `docker-compose.yml` se adaugă referința către `app.env` la secțiunile **frontend** și **backend** în blocul `env_file`, astfel cum se observă în exemplul de mai sus.

5. **Pornire și testare**
   - Se pornesc containerele rulând comanda:
     ```
     docker-compose up -d
     ```
   - Se verifică funcționarea aplicației accesând `http://localhost` în browser.  
   Dacă apare pagina de bază a nginx, se efectuează o reîncărcare pentru a porni serviciile necesare în mod corect.

## Întrebări și Răspunsuri

1. **În ce ordine sunt pornite containerele?**  
   Docker Compose pornește containerele relativ simultan, fără o ordine strictă prestabilită, deoarece în fișierul `docker-compose.yml` nu s-a specificat opțiunea `depends_on`. În practică, ordinea poate varia (ex. frontend, backend și apoi database) însă este important ca toate să fie operate pe aceeași rețea pentru a comunica corect.

2. **Unde sunt stocate datele bazei de date?**  
   Datele bazei de date sunt stocate în volumul `db_data`, care este montat în containerul de bază de date la calea `/var/lib/mysql`.

3. **Cum se numesc containerele proiectului?**  
   Numele containerelor, conform configurației din `docker-compose.yml`, sunt:
   - **frontend**
   - **backend**
   - **database**

4. **Cum se adaugă fișierul app.env pentru variabila de mediu APP_VERSION?**  
   Pentru a adăuga fișierul `app.env`, se realizează următorii pași:
   - Se creează un fișier nou în directorul `containers06` numit `app.env`.
   - În acest fișier se adaugă linia:  
     ```
     APP_VERSION=1.0.0
     ```
   - În fișierul `docker-compose.yml`, la serviciile **frontend** și **backend**, se adaugă secțiunea `env_file` cu referința `- app.env`, astfel încât ambele servicii să poată accesa această variabilă de mediu.

## Concluzii
Lucrarea demonstrează cum se poate crea și gestiona o aplicație PHP multi-container folosind Docker Compose. Prin configurarea fișierelor necesare (docker-compose.yml, fișierele de configurare pentru nginx, mysql.env și app.env) se obține o arhitectură modulară, în care fiecare container îndeplinește un rol specific în aplicatie. Această metodă facilitează scalabilitatea, întreținerea și dezvoltarea aplicațiilor, oferind totodată o bună separare a responsabilităților între serviciile de frontend, backend și baza de date.
