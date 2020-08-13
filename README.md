# Amusic
http://39.96.4.219:8080/

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).

### 模仿教学视频作品，练习一下Vue

### docker部署 (建议使用阿里云的镜像加速器)
```
docker build . -t amusic
docker run -d -p 8080:80 my-app
curl localhost:8080
```