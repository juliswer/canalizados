<template>
  <div>
    <client-only> 
    <div v-for="(item, index) in arrayGrupos" :key="index">
        <nuxt-link :to="{name: 'g-slug', params: {slug: item.slug}}">
           <div style="padding: 30px; box-shadow: -1px 1px 5px 1px rgb(152 163 179 / 50%); background: #FFFFFF 0% 0% no-repeat padding-box; border: 1px solid #7986CB; border-radius: 23px; opacity: 1; text-align: center;">
           <div @click="joinGroup">
             <b-button v-if="rolUser == 0" variant="link" v-b-popover.hover.top="'Toca el boton para unirte al grupo!'" style="position: absolute; right: 20px;">
               <i class="fas fa-plus-circle"></i>
             </b-button>
             <b-button v-if="rolUser == 3" variant="link" v-b-popover.hover.top="'Toca el boton para unirte al grupo!'" style="position: absolute; right: 20px;">
               <i class="fas fa-grip-lines"></i>
             </b-button>
           </div>
             <h2>{{item.titulo}}</h2>
             <p v-text="item.excerpt" v-if="item.excerpt"></p>
             <h4 class="cantPubliGrupo" v-if="item.cantPubli">{{item.cantPubli}} publicaciones </h4>
           </div>
         </nuxt-link>
         </div>
    </client-only>
  </div>
</template>


<script>

export default {
  components: { },
  name: "grupos",
          async fetch() {
       
         await this.$axios
               .$get("/grupos/getgrupostop")
               .then((response) => {
                   console.log(response)
                 this.arrayGrupos = response
               })
       
       
         await this.$axios
               .$get("/grupos/getslug?slug="+this.$route.params.slug+"&token="+this.$store.state.tokenUser)
               .then((response) => {
                 console.log(response)
                   if (response.status == 0) {
                   return app.redirect("/");
                   } else {
                   var rolName = 0;
                   if (response.grupo[0].rolUser == 1) {
                   rolName = "Owner";
                   }
                   if (response.grupo[0].rolUser == 2) {
                   rolName = "Moderador";
                   }
                   if (response.grupo[0].rolUser == 3) {
                   rolName = "Miembro";
                   }
       
                   this.responseGrupo= response
                   this.pGroup= response.grupo[0].id
                   this.tituloGrupo= response.grupo[0].titulo
                   this.excerptGrupo= response.grupo[0].excerpt
                   this.imagenGrupo= response.grupo[0].imagen
                   this.contenido= response.grupo[0].contenido
                   this.miembros= response.grupo[0].miembros
                   this.moderadores= response.grupo[0].moderadores
                   this.rolName= rolName
                   this.rolUser= response.grupo[0].rolUser
                   this.fechaM = response.grupo[0].fechaM
                   this.miembrosO = response.grupo[0].miembrosO
       
                   }
               })
       
         },
      data() {
    return {
       miembrosO: '',
        pGroup:'',
        fechaM: '',
        tituloGrupo: '',
        excerptGrupo: '',
        imagenGrupo: '',
        contenido: '',
        miembros: '',
        moderadores:'',
        rolName: '',
        rolUser: '', 
        tab: 0, 
        responseGrupo: '',
        arrayGrupos: []
    };
  },
  methods: {
      async  joinGroup(){
       if(this.rolUser == 2 || this.rolUser == 1){
          this.$router.push({ name: 'g-slug-configuracion-moderadores',  params: { slug: this.$route.params.slug }})
       }else{

      if(this.$store.state.cookieLogin){
   // console.log("hola join o leave")
          let formData = new FormData();
            
            formData.append('token', this.$store.state.tokenUser);
            formData.append('pGroup', this.pGroup);
            const response = await this.$axios.$post('/grupos/join/',
                formData,
                {
                headers: {
                    'Content-Type': 'multipart/form-data',
                }
              }
            )

      
            if(response.status == 0){
                
            }else{
               this.$emit("getGrupoNow");
              
            }
      }

       }
       
     
    },
    async followUser() {
      if (this.$store.state.cookieLogin == false) {
        this.$router.push("/iniciar-sesion");
        return false;
      }
      await this.$axios
        .$get(
          "/perfil/followunfollow?username=" +
            this.$route.params.username +
            "&token=" +
            this.$store.state.tokenUser
        )
        .then((response) => {
         // console.log(response);
          this.$emit("recargarDataUser");
        });
    },
    postNuevo() {
      if (this.$store.state.tokenUser != "") {
        this.$cookies.set(
          "group_cookie",
          {
            id: this.pGroup,
            imagenGrupo: this.imagenGrupo,
            tituloGrupo: this.tituloGrupo,
            slug: this.$route.params.slug,
            excerptGrupo: this.excerptGrupo,
          },
          {
            path: "/",
            maxAge: 60 * 60 * 24 * 7,
          }
        );
        this.$router.push({ name: "post-nuevo" });
      } else {
        this.$router.push({ name: "iniciar-sesion" });
      }
    },
      
    changeModeradores(moderadores){
        this.moderadores = moderadores
    },
    changeIconHeGroup(icono){
        this.imagenGrupo = icono
    },
      cambiarTab(tab){
          this.tab = tab
      },
      getGrupoNow(){
          this.refreshGroups()
      },
 
   refreshGroups() {
      this.$fetch()
    }
  },
  mounted() {
  },
};
</script>
 */