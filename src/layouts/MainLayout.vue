<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated class="bg-color-primary">
      <q-toolbar>
        <q-toolbar-title class="text-center"> 
          DigiTiket 
        </q-toolbar-title>
        </q-toolbar>
    </q-header>
    <q-page-container class="absolute-center">
     <q-btn @click="takePic()" size="75px" color="blue-grey-3" class="justify-center" >
        <q-icon name="photo_camera" size="130px"/>
        <br>
      <div class="text-weight-bold text-caption">Pulse para capturar imagen</div>
     </q-btn>
    </q-page-container>
    
    <q-dialog v-model="expanded"  >
      <q-card style="width: 100vw">
        <q-card-section class="row bg-primary items-center text-white">
          <div class="text-h6">Enviar por ...</div>
          <q-space />
          <q-btn flat icon="close" class="items-right text-white" @click="expanded = false"/>
        </q-card-section>
         <q-card-section>
          <!--q-input type="email" label="De" v-model="datosImagen.from"></q-input-->
          <!--q-input clearable type="email" label="Para" v-model="datosImagen.to">></q-input--->
          <q-input clearable label="Asunto" v-model="datosImagen.subject">></q-input>
          <q-input clearable text-area label="Texto" v-model="datosImagen.body" autogrow @keyup.enter.stop> ></q-input>
           <q-space />
        </q-card-section>
        <q-card-section> 
          <div class="justify-center text-center">
            <q-btn @click="sendMail()" icon-right="send" label="ENVIAR" color="primary"/>
          </div>
        </q-card-section>
        <q-card-section style="100vw">
          <q-img :src="datosImagen.dataURL+datosImagen.dataB64" />
        </q-card-section>
      </q-card>
    </q-dialog>
  </q-layout>
</template>

<script>
import { date } from 'quasar'
import { jsPDF } from "jspdf"

export default {
  data(){ 
    return { 
      options: {},
      expanded: false,
      datosImagen: { 
        // from: 'jvilata@vidawm.com',
        to: 'jvilata@edicom.es',
        subject: `digiticket-${date.formatDate(new Date(),'YYYY-MM-DD HHmmss')}`,
        body: '',
        data: '',
        dataB64: '',
        dataURL: '',
        oMyBlob: null
      }
    }
  },
  methods: { 
    takePic() { 
      this.options.destinationType = Camera.DestinationType.DATA_URL
      navigator.camera.getPicture(
            (data) => { // on success    
              this.datosImagen.contentType = 'image/jpeg'    
              this.datosImagen.dataB64 = data
              this.datosImagen.dataURL = 'data:' + this.datosImagen.contentType + ';base64,'
              this.expanded = true

              
              const img = new Image()
              img.src = this.datosImagen.dataURL+this.datosImagen.dataB64 // this.datosImagen.data
              img.onload = () => {
                const scaleFactor = 1
                // ( img.width > 600 ? 600.0 / img.width : 1)
                this.datosImagen.width = img.width * scaleFactor
                this.datosImagen.height = img.height * scaleFactor
                const elem = document.createElement('canvas')
                elem.width = this.datosImagen.width
                elem.height = this.datosImagen.height
                const ctx = elem.getContext('2d')
                // img.width and img.height will contain the original dimensions
                if (this.datosImagen.width >= this.datosImagen.height) { // rotate
                  var TO_RADIANS = Math.PI/180
                  elem.width = this.datosImagen.height
                  elem.height = this.datosImagen.width
                  ctx.translate(elem.width/2 , elem.height/2)
                  ctx.rotate( 90 * TO_RADIANS )
                  ctx.drawImage(img, -this.datosImagen.width/2, -this.datosImagen.height/2) // 0, 0, width, height)
                } else {
                  ctx.drawImage(img, 0, 0, this.datosImagen.width, this.datosImagen.height)
                }
                var data = ctx.canvas.toDataURL(this.datosImagen.contentType, 0.2)
                data = data.substring(this.datosImagen.dataURL.length)
                var raw = data // atob(this.datosImagen.dataB64)
                var rawLength = raw.length;
                var uInt8Array = new Uint8Array(rawLength)
                for (var i = 0; i < rawLength; ++i) {
                    uInt8Array[i] = raw.charCodeAt(i)
                }
                this.datosImagen.oMyBlob = new Blob([uInt8Array], {type: this.datosImagen.contentType})
                this.datosImagen.dataB64 = raw
              }
            },
            () => { // on fail
            },
            this.options)
    },
    sendMail(){
      const headerFormDataSinCredentials = {
        withCredentials: true,
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      }
      var nombre = `digiticket-${date.formatDate(new Date(),'YYYY-MM-DD HHmmss')}`

      var formData = new FormData()
      formData.append('action', 'sendattach')
      // formData.append('from', this.datosImagen.from)
      formData.append('to', this.datosImagen.to)
      formData.append('subject', (this.datosImagen.subject.length===0?nombre:this.datosImagen.subject))
      formData.append('body', `${this.datosImagen.body}\n\r\n\rPowered by DigiTicket. 2020`)
      //formData.append('attach', this.datosImagen.oMyBlob, nombre)

      this.$q.localStorage.set('to', this.datosImagen.to)
      // convierto a PDF
      var doc = new jsPDF()
      var imgData = this.datosImagen.dataURL+this.datosImagen.dataB64
      var ratio = this.datosImagen.height / this.datosImagen.width
      if (this.datosImagen.width > this.datosImagen.height) ratio = this.datosImagen.width / this.datosImagen.height
      var w = doc.internal.pageSize.getWidth()
      var h = w * ratio
      var dif = h - doc.internal.pageSize.getHeight()
      if (dif > 0) {
        w = w - dif
        h = w * ratio
      }
      doc.addImage(imgData, 'JPEG', 10, 10,Math.round(w)-20,Math.round(h)-20, '', 'MEDIUM') // this.datosImagen.width, this.datosImagen.height)
      var dataPDF = doc.output('dataurlstring')  // data:application/pdf;filename=generated.pdf;base64,
      var pos = dataPDF.indexOf(';base64,')+(';base64,').length
      dataPDF = 'data:application/pdf;base64,' + dataPDF.substring(pos)
      // this is the complete list of currently supported params you can pass to the plugin (all optional)
      var options = {
        to: this.datosImagen.to,
        message: `${this.datosImagen.body}\n\r\n\rPowered by DigiTicket. 2020`, // not supported on some apps (Facebook, Instagram)
        subject: (this.datosImagen.subject.length===0?nombre:this.datosImagen.subject), // fi. for email
        files: [dataPDF], // an array of filenames either locally or remotely
        // files: [this.datosImagen.dataURL+this.datosImagen.dataB64], // an array of filenames either locally or remotely
        // url: 'https://www.website.com/foo/#bar?a=b',
        chooserTitle: 'Elija una app', // Android only, you can override the default share sheet title
        // appPackageName: 'com.apple.social.facebook', // Android only, you can provide id of the App you want to share with
        // iPadCoordinates: '0,0,0,0' //IOS only iPadCoordinates for where the popover should be point.  Format with x,y,width,height
      };
      var vthis = this
      var onSuccess = function(result) {
        vthis.$q.dialog({ title: 'OK', message: 'Ticket subido correctamente a ' + result.app })
        .onDismiss(()=>{
              vthis.expanded = false
            })
      };

      var onError = function(msg) {
        vthis.$q.dialog({ title: 'Error', message: 'Error al subir ticket.'+msg+'. Vuélvalo a intentar o contacte con el administrador' })
      };

      window.plugins.socialsharing.shareWithOptions(options, onSuccess, onError);

      /* this.$axios.post('lib/sendmail.php', formData, headerFormDataSinCredentials)
        .then(response => {
          if (response.data.success) {
            this.$q.dialog({ title: 'OK', message: 'Ticket subido correctamente' })
            .onDismiss(()=>{
              this.expanded = false
            })
          } else {
            this.$q.dialog({ title: 'Error', message: 'Error al subir ticket. Vuélvalo a intentar o contacte con el administrador' })
          }
        })
        .catch(error => {
          this.$q.dialog({ title: 'Error', message: 'Error al subir ticket. Vuélvalo a intentar o contacte con el administrador' })
        })   */
    }
  },
  mounted () {
    this.datosImagen.from = this.$q.localStorage.getItem('from')
    // this.datosImagen.subject = this.$q.localStorage.getItem('subject')
    // this.datosImagen.body = this.$q.localStorage.getItem('body')
  }
}
</script>
