# CDN Vuetify
Accede al siguiente enlace: https://vuetifyjs.com/es-MX/getting-started/quick-start#uso-de-una-cdn
```
<!DOCTYPE html>
<html>
<head>
  <link href="https://fonts.googleapis.com/css?family=Roboto:100,300,400,500,700,900" rel="stylesheet">
  <link href="https://cdn.jsdelivr.net/npm/@mdi/font@4.x/css/materialdesignicons.min.css" rel="stylesheet">
  <link href="https://cdn.jsdelivr.net/npm/vuetify@2.x/dist/vuetify.min.css" rel="stylesheet">
  <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no, minimal-ui">
</head>
<body>
  <div id="app">
    <v-app>
      <v-content>
        <v-container>Hello world</v-container>
      </v-content>
    </v-app>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/vue@2.x/dist/vue.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/vuetify@2.x/dist/vuetify.js"></script>
  <script>
    new Vue({
      el: '#app',
      vuetify: new Vuetify(),
    })
  </script>
</body>
</html>
```

## CDN Firebase
Accede al siguiente enlace: https://firebase.google.com/docs/web/setup?authuser=0#delay-sdks-cdn
```
<!-- Firebase App (the core Firebase SDK) is always required and must be listed first -->
<script defer src="https://www.gstatic.com/firebasejs/7.5.2/firebase-app.js"></script>
<script defer src="https://www.gstatic.com/firebasejs/7.5.2/firebase-firestore.js"></script>

<script>
var firebaseConfig = {
  apiKey: "api-key",
  authDomain: "project-id.firebaseapp.com",
  databaseURL: "https://project-id.firebaseio.com",
  projectId: "project-id",
  storageBucket: "project-id.appspot.com",
  messagingSenderId: "sender-id",
  appId: "app-id",
  measurementId: "G-measurement-id",
};

// Initialize Firebase
firebase.initializeApp(firebaseConfig);
var db = firebase.firestore();
</script>
```

## jsonplaceholder
Agregaremos los siguientes datos: https://jsonplaceholder.typicode.com/
```
methods: {
  agregarDatos(){
    fetch('posts.json').then(res => res.json()).then(res => {
      console.log(res);
      res.forEach(item => {
        // console.log(item);
        db.collection('blog').add(item)
      })
    })
  }
}
```
## boton
```
<v-btn @click="agregarDatos">Agregar Datos</v-btn>
```

## Limit Firestore
https://firebase.google.com/docs/firestore/query-data/order-limit-data?authuser=0
```
data: {
  entradas: [],
  last: null,
  desactivar: false
},
created() {
  this.leerDatos();
},
methods: {
  leerDatos() {
    db.collection("blog")
      .limit(2)
      .orderBy("id")
      .get()
      .then(querySnapshot => {
        // Nos guarda el último artículo
        this.last = querySnapshot.docs[querySnapshot.docs.length - 1];
        
        querySnapshot.forEach(doc => {
          this.entradas.push(doc.data());
        });

      });
  },
}
<v-container>
  <v-row>
    <v-col
      cols="12"
      md="4"
      v-for="(item, index) in entradas"
      :key="index"
    >
      <v-card>
        <v-card-title>{{item.title}}</v-card-title>
        <v-card-subtitle>
          {{item.id}}
        </v-card-subtitle>
        <v-card-text>{{item.body}}</v-card-text>
      </v-card>
    </v-col>
  </v-row>
</v-container>
```
## Paginación

https://firebase.google.com/docs/firestore/query-data/query-cursors?authuser=0#paginate_a_query
```
siguiente() {
  db.collection("blog")
    .limit(2)
    .orderBy("id")
    .startAfter(this.last)
    .get()
    .then(querySnapshot => {
      this.last = querySnapshot.docs[querySnapshot.docs.length - 1];
      querySnapshot.forEach(doc => {
        this.entradas.push(doc.data());
      });
    })
    .then(() => {
      db.collection("blog")
        .limit(2)
        .orderBy("id")
        .startAfter(this.last)
        .get()
        .then(query => {
          if (query.empty) {
            this.desactivar = true;
          }
        });
    });
}
<v-container>
  <v-btn @click="siguiente" :disabled="desactivar">Más artículos...</v-btn>
</v-container>
```

## Paginación 2.0
Veamos como agregar mejoras

Utilizaremos la paginación de Vuetify: https://vuetifyjs.com/es-MX/components/paginations#long

## Agregar Componente:
```
<!-- Paginación Vuetify -->
<div class="text-center">
  <v-container>
    <v-row justify="center">
      <v-col cols="8">
        <v-container class="max-width">
          <v-pagination
            v-model="page"
            class="my-4"
            :length="paginas"
            @input="onPageChange"
          ></v-pagination>
        </v-container>
      </v-col>
    </v-row>
  </v-container>   
</div>
#Agregar data:
data:{
  entradas: [],
  total: 0,
  paginas: 0,
  porPagina: 6,
  page: 1
},
```


## Methods:
```
methods: {
  leerDatos(){

    db.collection('blog').get().then(res => {
      this.total = res.size
      this.paginas = Math.ceil((this.total / this.porPagina)) // Redondea hacia arriba
    })

    db.collection('blog')
      .limit(this.porPagina)
      .orderBy('id')
      .get()
      .then(query => {
        query.forEach(item => {
          this.entradas.push(item.data())
        })
      })
  },
  onPageChange(){
    db.collection('blog')
      .limit(this.porPagina)
      .orderBy('id')
      .startAfter(this.porPagina*(this.page-1))
      .get()
      .then(query => {
        this.entradas = []
        query.forEach(item => {
          this.entradas.push(item.data())
        })
      })
  }
},
```

## Filtros optativos:
```
filters:{
  limite(value){
    return value.substring(0, 90);
  },
  limiteTitulo(value){
    return value.substring(0, 20);
  }
}
<v-container>
  <v-row>
    <v-col cols="12" md="4" v-for="(item, index) in entradas" :key="index">
      <v-card>
        <v-card-title>{{item.title | limiteTitulo}}</v-card-title>
        <v-card-subtitle>{{item.id}}</v-card-subtitle>
        <v-card-text>{{item.body | limite}}</v-card-text>
      </v-card>
    </v-col>
  </v-row>
</v-container>
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).


