# pokeapp

Crear repositorio en GitHub

Nuevo repo público llamado pokeapp.

Clonar localmente:

git clone git@github.com:TU_USUARIO/pokeapp.git
cd pokeapp


Inicializar proyecto Node.js

mkdir src
cd src
npm init -y
npm install express axios


En src/index.js:

const express = require('express');
const axios = require('axios');
const app = express();
const PORT = process.env.PORT || 3000;

app.use(express.static('public'));

app.get('/api/pokemon', async (req, res) => {
  try {
    const { data } = await axios.get('https://pokeapi.co/api/v2/pokemon/ditto');
    // Procesar solo lo necesario
    res.json({
      name: data.name,
      sprite: data.sprites.front_default,
      abilities: data.abilities.map(a => a.ability.name)
    });
  } catch (e) {
    res.status(500).json({ error: 'Falló la consulta' });
  }
});

app.listen(PORT, () => console.log(`Server en puerto ${PORT}`));


Crear carpeta src/public con un index.html muy básico:

<!DOCTYPE html>
<html>
<head><meta charset="utf-8"><title>PokéApp</title></head>
<body>
  <h1>Ditto</h1>
  <div id="info"></div>
  <script>
    fetch('/api/pokemon')
      .then(r => r.json())
      .then(p => {
        document.getElementById('info').innerHTML = `
          <img src="${p.sprite}" alt="${p.name}">
          <p>Habilidades: ${p.abilities.join(', ')}</p>
        `;
      });
  </script>
</body>
</html>


Dockerizar la aplicación
En la raíz (pokeapp/Dockerfile):


FROM node:18-alpine
WORKDIR /app
COPY src/package*.json ./
RUN npm install --production
COPY src ./
EXPOSE 3000
CMD ["node", "index.js"]


Configurar GitHub Actions
En .github/workflows/docker-publish.yml:

name: CI/CD to DockerHub

on:
  push:
    branches: [ main ]

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build image
        run: docker build -t ${{ secrets.DOCKERHUB_USER }}/pokeapp:${{ github.sha }} .
      - name: Login DockerHub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USER }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      - name: Push image
        run: docker push ${{ secrets.DOCKERHUB_USER }}/pokeapp:${{ github.sha }}


Secrets en GitHub:
DOCKERHUB_USER
DOCKERHUB_TOKEN (token de acceso).


Preparar EC2 (Ubuntu)
Lanzar instancia t2.micro, abrir puerto 3000 en Security Group.

Conectar via SSH:

sudo apt update
sudo apt install -y docker.io
sudo usermod -aG docker $USER

(Cerrar sesión y volver a entrar para permisos Docker).



Desplegar desde EC2
Manual (ó automatizable con un step extra en GH Actions):

docker pull TU_USUARIO/pokeapp:latest   # o usar tag SHA
docker run -d --name pokeapp -p 3000:3000 TU_USUARIO/pokeapp:latest

Comprobar en el navegador: http://<EC2_PUBLIC_IP>:3000



URLs finales
Repositorio GitHub:
https://github.com/TU_USUARIO/pokeapp
DockerHub:
https://hub.docker.com/r/TU_USUARIO/pokeapp
App en EC2:
http://<EC2_PUBLIC_IP>:3000


En gitbash
chmod 600 /c/Users/jakil/Desktop/pokeapp-key.pem
ssh -i /c/Users/jakil/Desktop/pokeapp-key.pem ubuntu@X.X.X.X

