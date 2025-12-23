# LWT_Docker
Use Docker to create an LWT (Learning-with-texts) website. 

This docker setup will allow you to skip the installation of a full webserver and database on your machine. 

1) Install docker and docker-compose for your operating system. (https://docs.docker.com/install/)
2) Download the Dockerfile and docker-compose.yml and connect.inc.php files from this repository and put into a directory.
3) Create a sub-directory called lwt_html
4) Download and extract lwt into the lwt_html directory (https://sourceforge.net/projects/learning-with-texts/)
5) Copy the connect.inc.php into the lwt_html directory.
6) Run docker-compose:
  <BR>docker-compose build lwt
  <BR>docker-compose up lwt
7) Access your LWT website on: http://localhost/

## Quick start on new hardware

1. Install Docker and Docker Compose v2+.
2. From the project root run:
   - `docker compose build`
   - `docker compose up -d`
3. Open the app at http://localhost:9090 (port mapped in `docker-compose.yml`).
4. MySQL runs in the `db` service (image `mysql:5.7`) with database `lwt` and root password `root`; data persists in the `db_data` volume.
5. Stop with `docker compose down` (add `-v` if you want to drop the database volume).
