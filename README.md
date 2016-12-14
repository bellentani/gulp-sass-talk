#Gulp + Sass: automatize a compilação do seu CSS

Gulp é um task runner. O que isso quer dizer? Traduzindo livremente ele é um “automatizador de tarefas”. Ele executa tarefas repetitivas de seu cotidiano de trabalho (workflow) - muitas vezes de forma automática, o que economiza retrabalho e gera ganho de tempo.

É uma prática já antiga e utilizada por caras como *Ant* (para Apache), *Phing* (para PHP) e *Rake* (para Ruby) - e também pelo primeiro "rockstar" dessa era Javascript, o **Grunt**.

##Gulp vs. Grunt: ROUND 1 FIGHT!

Existem dois task runners muito populares atualmente, o [Gulp](http://gulpjs.com/) e o [Grunt](http://gruntjs.com/), cada um com suas características próprias.

O **Grunt** foi o primeiro que mexi (e foi recentemente, primeiro contato em 2013) e o primeiro a ganhar popularidade: ele permitia que você automatizasse tarefas como o pré-processamento de CSS com [Less](http://lesscss.org/) e [Sass](http://sass-lang.com/), mas também com uma diversidade enorme de outras tarefas (concatenar, minificar e versionar Javascript, por exemplo). Sua sintaxe é baseada em uma estrutura JSON e pode ser mesclada com Javascript, o que deixa a sua configuração meio confusa em workflows complexos porque exige um conhecimento profundo do que está acontecendo em cada parte do seu arquivo de configuração.

O **Gulp** veio comendo pelas bordas, mas com um monte de material já “no pacote” que antes precisava ser instalado individualmente e dava um certo trabalho para manter atualizado - se comparado ao Grunt. Sem contar que Gulp tem uma performance maior porque usa um sistema de “pipelines” que forma filas e permite a criação de workflows mais complexos - e sua sintaxe é muito mais parecida com o que a gente usa em nossos Javascripts cotidianos (com callbacks e afins).

Dentro do pacote do Gulp a gente já vem três coisas essenciais:

* Você define tarefas para o gulp com o `gulp.task()`
* Abre, acessa ou define arquivos e diretórios através de `gulp.src()`
* Define o diretório de destino do output com `gulp.dest()`
* Vigia modificações no file system com `gulp.watch()`

##Instalação

###Pré-requisitos

* Ter o *Node.js* e o *NPM* instalados (se não tiver, só baixar no site do [Node.js](https://nodejs.org/en/download/) e instalar);
* Opcional: conhecer um pouco da arquitetura do `package.json`

Instale o **Gulp** globalmente no seu console através do comando:

```{r, engine='bash'}
$ npm install -g gulp
```

Se você não tiver um arquivo `package.json` modelo, você pode iniciar um através do comando:

```{r, engine='bash'}
$ npm init
```
Em seguida responda as perguntas (a maioria delas pode ser deixada em branco, basta clicar <ENTER> ou digitar <Yes>.

Para  instalar localmente e salvar em seu `package.json`:

```{r, engine='bash'}
$ npm install --save-dev gulp
```

##O nosso workflow

Nós vamos apenas gerar arquivos CSS utilizando Sass e minificá-los já com a geração de um arquivo Sourcemap para permitir que o debuguemos em ambientes de homologação.

O workflow será:

1. Encontrar os arquivos que queremos pré-processar (no caso, os arquivos com extensão Sass e SCSS);
* Mapear o(s) arquivo(s) a ser(em) gerado(s);
* Gerar o CSS baseado no Sass
* Minificar o arquivo gerado
* Escrever o arquivo Sourcemap respectivo na mesma pasta do CSS gerado
* Salvar o arquivo CSS
* Dar reload no navegador automaticamente após gerar o CSS

Para isso vamos utilizar os pacotes `gulp-sass`, `gulp-sourcemaps` e `browser-sync`.
Para instalar eles podemos fazer um a um:

```{r, engine='bash'}
$ npm install --save-dev gulp-sass
```

```{r, engine='bash'}
$ npm install --save-dev gulp-sourcemaps
```

```{r, engine='bash'}
$ npm install --save-dev browser-sync
```

Ou instalar todos de uma vez:

```{r, engine='bash'}
$ npm install --save-dev gulp-sass gulp-sourcemaps browser-sync
```

O nosso workflow vai se formar com três tasks principais:

1. Iremos utilizar a gulp sass para gerar o Sass;
* Configurar o reload do automático do navegador usando o browserSync;
* E a gulp watch para ficar “vigiando” modificações em arquivos Sass no nosso diretório.

A nossa estrutura de pasta será:

```
|_dist (os arquivos gerados)
  |_css (os arquivos CSS processados)
|_src (os arquivos de desenvolvimento)
  |_sass (os arquivos Sass para serem processados)

```

Crie um arquivo `gulpfile.js na raiz do seu projeto. Teremos o seguinte conteúdo inicial chamando os módulos:

```javascript
var gulp = require('gulp'),
sass = require('gulp-sass'),
sourcemaps = require('gulp-sourcemaps'),
browserSync = require('browser-sync').create();
```

Depois, incluiremos os nossos caminhos utilizando uma notação de objeto:

```javascript
var config = {
  srcPath: 'src/',
  distPath: 'dist/'
};
```

Depois iremos criar as tarefas base para registrá-las apenas:

```javascript
gulp.task('browserSync', function(){
  console.log('browserSync');
});

gulp.task('sass', function(){
  console.log('sass');
});

gulp.task('watch', function(){
  console.log('watch');
});
```

Teste elas em seu console chamando:

```{r, engine='bash'}
gulp sass
```

```{r, engine='bash'}
gulp watch
```

```{r, engine='bash'}
gulp browserSync
```

O resultado será a impressão de mensagens em seu console das três tarefas na ordem que você as chamou.

Na sequência vamos cadastrar:

```javascript
gulp.task('browserSync', function() {
  browserSync.init({ //inicia o server do browsersync
    server: {
      baseDir: './', //define o diretório base
    },
    port: 8080, //define a porta, você pode mudar para a que quiser
    startPath: 'index.html', //define qual é o arquivo que abrirá como padrão quando ele iniciar
  })
});

gulp.task('sass', function(){
  //a marcação define que iremos pegar todos os arquivos SCSS e Sass da pasta src/sass, inclusive subpastas e seu conteúdo se houverem
  return gulp.src(config.srcPath+'sass/**/*.+(scss|sass)')
    .pipe(sourcemaps.init()) //iniciamos o sourcemap para gravar o MAP para debuging
    .pipe(sass({ //iniciamos o modulo do Sass
      outputStyle: 'compressed' //adicionamos a opção para que o produto final seja comprimido/minificado
    }).on('error', sass.logError)) // Se der erro exibirá um log e criará um arquivo para análise
    .pipe(sourcemaps.write('./')) //Escrevera o sourcemap na mesma pasta ou subpasta do CSS gerado
    .pipe(gulp.dest(config.distPath+'css')) //Irá salvar o arquivo na estrutura equivalente dentro da pasta /dist/css
    .pipe(browserSync.reload({ //ativa o reload da pagina quando terminar de fazer o Sync
      stream: true // (se o servidor estiver iniciado ele dá o reload)
    }));
});

gulp.task('watch', ['browserSync'], function() { //inicia a watch, colocando junto com ela a browserSync para iniciar o server toda vez que a watch executar
    gulp.watch(config.srcPath+'sass/**/*.+(scss|sass)', ['sass']); //vigia os arquivos SCSS e Sass na pasta /src/sass, quando um for modificado ele executa a task SASS e ativa o Browsync (por causa da option stream ativa na task [sass])
});
```

O arquivo inteiro ficará algo como:

```javascript
var gulp = require('gulp'),
sass = require('gulp-sass'),
sourcemaps = require('gulp-sourcemaps'),
browserSync = require('browser-sync').create();

var config = {
  srcPath: 'src/',
  distPath: 'dist/'
};

gulp.task('browserSync', function() {
  browserSync.init({
    server: {
      baseDir: './',
    },
    port: 8080,
    startPath: 'index.html',
  })
});

gulp.task('sass', function(){
  return gulp.src(config.srcPath+'sass/**/*.+(scss|sass)')
    .pipe(sourcemaps.init())
    .pipe(sass({
      outputStyle: 'compressed'
    }).on('error', sass.logError)) // Using gulp-sass
    .pipe(sourcemaps.write('./'))
    .pipe(gulp.dest(config.distPath+'css'))
    .pipe(browserSync.reload({
      stream: true
    }));
});

gulp.task('watch', ['browserSync'], function() {
    gulp.watch(config.srcPath+'sass/**/*.+(scss|sass)', ['sass']);
});
```

##Testando o resultado

Para testar faça algumas coisas:

1. Abra o arquivo `index.html` e o edite;
2. Em seguida coloque o texto que quiser;
3. Execute a task `gulp watch`;
4. Modifique o arquivo `src/sass/main.scss` e modifique a `background-color` para `green`;
5. Veja a magia acontecer em seu navegador.

##O mais legal de tudo
Esse workflow não precisa do Sass instalado via Ruby na sua máquina. Ele usa um módulo "portado" para Node.js chamado `node-sass`.

Para utilizar o Compass, por exemplo, você pode fazer uma pequena alteração nesse workflow e, desde que tenha o Ruby, Sass e Compass instalados na sua máquina poderá utilizar dessa biblioteca Sass sem problema algum.

Se tiver o Ruby + Sass + Compass na sua máquina, só instalar o módulo para gulp:

```{r, engine='bash'}
$ npm install --save-dev gulp-compass
```

Crie uma task [compass]:

```javascript
gulp.task('compass', function() {
  gulp.src(config.srcPath+'sass/**/*.+(scss|sass)')
    .pipe(compass({
      css: config.distPath+'css/',
      sass: config.srcPath+'sass/',
      style: 'compressed',
      sourcemap: true
    }))
    .on('error', function(error) {
      console.log(error);
      this.emit('end');
    })
    .pipe(gulp.dest(config.distPath+'css'))
    .pipe(browserSync.reload({
      stream: true
    }));
});
```
Observe que a estrutura mudou bastante e foi removida o módulo `sourcemap`.

Isso se dá porque o `gulp-compass` já faz a geração do Sourcemap quando realiza a compressão se você assim definir em suas options.

Aí você pode criar uma task também nova para o Watch especificamente para casos de uso do Compass:

```javascript
gulp.task('watch-compass', ['browserSync'], function() {
    gulp.watch(config.srcPath+'sass/**/*.+(scss|sass)', ['compass']);
});
```

Rode a tarefa `gulp watch-compass` e execute o teste. 

Voilá, está funcionando (ou era pra estar)! :)

O arquivo ficará da seguinte maneira:

```javascript
var gulp = require('gulp'),
sass = require('gulp-sass'),
compass = require('gulp-compass'),
sourcemaps = require('gulp-sourcemaps'),
browserSync = require('browser-sync').create();

var config = {
  srcPath: 'src/',
  distPath: 'dist/'
};

gulp.task('browserSync', function() {
  browserSync.init({
    server: {
      baseDir: './',
    },
    port: 8080,
    startPath: 'index.html',
  })
});

gulp.task('sass', function(){
  return gulp.src(config.srcPath+'sass/**/*.+(scss|sass)')
    .pipe(sourcemaps.init())
    .pipe(sass({
      outputStyle: 'compressed'
    }).on('error', sass.logError)) // Using gulp-sass
    .pipe(sourcemaps.write('./'))
    .pipe(gulp.dest(config.distPath+'css'))
    .pipe(browserSync.reload({
      stream: true
    }));
});

gulp.task('compass', function() {
  gulp.src(config.srcPath+'sass/**/*.+(scss|sass)')
    .pipe(compass({
      css: config.distPath+'css/',
      sass: config.srcPath+'sass/',
      style: 'compressed',
      sourcemap: true
    }))
    .on('error', function(error) {
      console.log(error);
      this.emit('end');
    })
    //.pipe(minifyCSS())
    .pipe(gulp.dest(config.distPath+'css'))
    .pipe(browserSync.reload({
      stream: true
    }));
});

gulp.task('watch', ['browserSync'], function() {
    gulp.watch(config.srcPath+'sass/**/*.+(scss|sass)', ['sass']);
});

gulp.task('watch-compass', ['browserSync'], function() {
    gulp.watch(config.srcPath+'sass/**/*.+(scss|sass)', ['compass']);
});
```

##Dúvidas, bugs e sugestões

Basta criar uma [issue](https://github.com/bellentani/gulp-sass-talk/issues) que a gente discute e entramos em contato. Caso queira conversar sobre qualquer assunto relacionado entre em contato através do [meu site](www.bonsaiux.com.br).
