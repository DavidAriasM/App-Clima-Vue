<template>
  <div class="contenedor">
      <div v-if="errorEstado">{{ mensajeError }}</div>
      <div class="tiempo-hoy">
          <div class="caja1" id="caja1">
              <h1 id="temperatura-valor">{{ temperaturaValor }}°C</h1>
              <h1 id="temperatura-descripcion"> {{ temperaturaDescripcion }}</h1>
          </div>
          <div id="caja2">
                <h2 id="ubicacion">{{ temperaturaUbicacion }}</h2>
                <img id="icono-animado" :src="icono" alt="icono-clima">
            </div>
            <div id="caja3">
                <h3>Veloc. del Viento</h3>
                <h1 id="viento-velocidad">{{ vientoVelocidad }}</h1>
            </div>
            <form @submit.prevent="buscarCiudad(inputBuscar)" id="formBuscador">
                <div class="buscador">
                    <input type="text" class="input-buscador" name="input-buscador" id="input-buscador" v-model.trim="inputBuscar" placeholder="Ingrese una ciudad a buscar...">
                    <button class="btn-buscador" onclick=""><i class="fa-solid fa-magnifying-glass"></i></button>
                </div>
            </form>
      </div>
      <div class="tiempo-dias-siguientes">
            <div class="tiempo-dias" id="tiempo-dias-1"  v-for="(dia, index) in diasSiguientes" :key="index">
                <p class="fecha">{{ dia.horaTiempo }}</p>
                <p class="temp">{{ dia.tempTiempo}}°C</p>
                <img id="iconoDiasSgtes" :src="icono" alt="icono del clima" height="80" width="80">
            </div>
        </div>
  </div>
</template>

<script>
export default {
    data() {
        return {
            inputBuscar: '',
            urlApiInicio: 'https://api.openweathermap.org/data/2.5/',
            apiKey: process.env.VUE_APP_API_KEY,
            longitud: '',
            latitud: '',
            temperaturaValor: '',
            temperaturaDescripcion: '',
            temperaturaUbicacion: '',
            vientoVelocidad: '',
            icono: '',
            err: '',
            errorEstado : false,
            mensajeError : '',
            diasSiguientes: []
        }
    },
    methods:{
        buscarCiudad: function(ciudad){
            let urlBuscar = `${this.urlApiInicio}weather?q=${ciudad}&lang=Es&units=metric&appid=${this.apiKey}`

                fetch(urlBuscar)
                .then(response => response.json())
                .then(data => {
                     this.rellenarValor(data);
                    this.cargarIconos(data.weather[0].description)
                    this.inputBuscar = '';
                
                    const urlDias = `${this.urlApiInicio}forecast?q=${ciudad}&lang=Es&units=metric&appid=${this.apiKey}`

                    fetch(urlDias)
                    .then(response => response.json())
                    .then(data => {
                        this.cargarDiasSiguientes(data);
                    })
                    .catch(error => { console.log(error) });
                    })

                .catch (error => {
                console.log(error);
                alert('No podemos obtener información de esta ciudad, pruebe con otra');
                this.inputBuscar = '';
                });            
        },
        position: function(location){
            this.longitud = location.coords.longitude;
            this.latitud = location.coords.latitude;

            const url = `${this.urlApiInicio}weather?lat=${this.latitud}&lon=${this.longitud}&lang=Es&units=metric&appid=${this.apiKey}`;
            
                fetch(url)
                    .then(response => response.json())
                    .then(data => {
                        this.rellenarValor(data);
                        this.cargarIconos(data.weather[0].description)
                    })
                .catch(error => { console.log(error) });

            const urlDias = `${this.urlApiInicio}forecast?lat=${this.latitud}&lon=${this.longitud}&lang=Es&units=metric&appid=${this.apiKey}`;

                fetch(urlDias)
                    .then(response => response.json())
                    .then(data => {
                        this.cargarDiasSiguientes(data);
                    })
                .catch(error => { console.log(error) });
        },
        error: function(err){
            console.log(err.message);
            switch(err.code){
            case err.PERMISSION_DENIED:
                //Si no se permite obtener la ubicación desde su navegador por defecto cargará la temperatura de Santiago
                const urlFija = `${this.urlApiInicio}weather?q=Santiago&lang=Es&units=metric&appid=${this.apiKey}`
                console.log(urlFija)

                fetch(urlFija)
                    .then(respuesta => { return respuesta.json() })
                    .then(data => {
                        this.rellenarValor(data)
                        this.cargarIconos(data.weather[0].description)
                    })
                    .catch(error => {
                        console.log(error);
                    })

                const urlDias = `${this.urlApiInicio}forecast?q=Santiago&lang=Es&units=metric&appid=${this.apiKey}`

                fetch(urlDias)
                    .then(respuesta => { return respuesta.json() })
                    .then(data => {
                        this.cargarDiasSiguientes(data);

                    })
                    .catch(error => { console.log(error); })

            break;
            case err.POSITION_UNVAILABLE:
                this.mensajeError = "Tu posicion actual no está disponible";
                break;
            case err.TIMEOUT:
                this.mensajeError = "No hemos podido obtener tu ubicación en un tiempo prudencial"
                break;
            default:
                this.mensajeError = "Error Desconocido"
                break
            }
            this.errorEstado = true;

        },
        rellenarValor: function(data){
            this.temperaturaDescripcion = data.weather[0].description;
            this.temperaturaValor = Math.round(data.main.temp);
            this.temperaturaUbicacion = data.name;
            this.vientoVelocidad = data.wind.speed;
        },
        cargarDiasSiguientes: function(data){
            this.diasSiguientes = []
            for (let i = 0; i < data.list.length; i++) {
                if (parseInt(data.list[i].dt_txt[11]) === 1 && parseInt(data.list[i].dt_txt[12]) === 8) {
                    this.diasSiguientes.push({
                        horaTiempo : this.covertirFecha(data.list[i].dt_txt),
                        tempTiempo : Math.round(data.list[i].main.temp),
                        descTiempo : data.list[i].weather[0].description,
                        icono      : this.cargarIconos(data.list[i].weather[0].description)
                    })
                }
            }
        },
        covertirFecha: function(fecha) {
            let filtro = fecha.split(' ')
            filtro = filtro[0].split('-')
            return filtro[2] + '/' + filtro[1]
        },
        cargarIconos: function(data){
            let nubes = "/img/iconos/cloudy-day-2.svg"
            let cieloClaro = '/img/iconos/day.svg'
            let chubascos = "/img/iconos/rainy-2.svg"
            let muyNuboso = "/img/iconos/cloudy.svg"
            let lluviaLigera = "/img/iconos/rainy-5.svg"
            let nieve = '/img/iconos/snowy-5.svg'

            if (data === 'cielo claro') {
                this.icono = cieloClaro;
            } else if (data === 'nubes' || data === 'algo de nubes') {
                this.icono = nubes
            } else if (data === 'muy nuboso' || data === "nubes dispersas") {
                this.icono = muyNuboso
            } else if (data === 'chubascos' || data === 'chubasco de ligera intensidad') {
                this.icono = chubascos
            } else if (data === 'lluvia ligera' || data === 'lluvia de gran intensidad') {
                this.icono = lluviaLigera
            } else if (data === 'nieve' || data === 'nevada ligera') {
                this.icono = nieve;
            }
        }
    },
    created(){
        if("geolocation" in navigator){
            navigator.geolocation.getCurrentPosition(position => { this.position(position) }, err => { this.error(err) })
        }else{
            console.log('No se pudo obtener ubicación del dispositivo');
        }
    }
}
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Abel&display=swap');
    * {
        margin: 0;
        padding: 0;
        box-sizing: border-box;
    }
    
    body {
        height: 100%;
        font-family: 'Abel', sans-serif;
    }
    
    .contenedor {
        width: 100%;
        height: 100vh;
        display: flex;
        flex-direction: flex;
        justify-content: center;
        align-items: center;
        background: linear-gradient(to bottom right, rgb(184, 249, 252), rgb(117, 201, 250));
        padding: 100px;
        transition: .5s;
    }
    
    .mensaje-error {
        display: none;
    }
    
    .tiempo-hoy {
        width: 50%;
        min-height: 700px;
        background-color: rgb(241, 241, 241);
        border-top-left-radius: 5px;
        border-bottom-left-radius: 5px;
        background: url(/img/Atardecer1.jpg) no-repeat center;
        background-size: cover;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        text-align: center;
        font-family: 'Abel', sans-serif;
        color: rgba(255, 255, 255, 0.7);
        white-space: nowrap;
        overflow-x: hidden;
    }
    
    .caja1 {
        margin-top: 50px;
    }
    
    .caja1 #temperatura-valor {
        font-size: 70px;
    }
    
    .titulo-proximos-dias {
        height: 0;
        text-align: center;
    }
    
    .tiempo-dias-siguientes {
        width: 30%;
        min-height: 700px;
        background: #fff;
        border-top-right-radius: 5px;
        border-bottom-right-radius: 5px;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
    }
    
    .tiempo-dias {
        width: 100%;
        min-height: 140px;
        font-size: 40px;
        font-family: 'Abel', sans-serif;
        display: flex;
        justify-content: center;
        align-items: center;
        line-height: 70px;
    }
    .fecha {
        font-size: 20px;
        height: 50px;
    }
    
    .temp {
        height: 50px;
    }
    
    .icono {
        height: 50px;
    }

    #icono-animado{
        width: 200px;
        height: 200px;
    }
    
    .buscador {
        margin-bottom: 10px;
    }
    
    .input-buscador {
        width: 250px;
        height: 30px;
        border-top-left-radius: 5px;
        border-bottom-left-radius: 5px;
        border: none;
        text-align: center;
        text-transform: uppercase;
        font-size: 12px;
    }
    
    .btn-buscador {
        width: 60px;
        height: 30px;
        border: none;
        border-top-right-radius: 5px;
        border-bottom-right-radius: 5px;
        margin-left: -5px;
        cursor: pointer;
        transition: .5s;
    }
    
    .btn-buscador:hover {
        background-color: #3498db;
        color: #fff;
    }
    
    @media(max-width:1100px) {
        .tiempo-dias-siguientes {
            width: 30%;
        }
    }
    
    @media (max-width:900px) {
        .contenedor {
            flex-direction: column;
            padding: 50px;
        }
        .tiempo-hoy {
            width: 100%;
            margin-bottom: 0;
            min-height: 30%;
            border-top-left-radius: 5px;
            border-top-right-radius: 5px;
            border-bottom-left-radius: 0px;
            overflow-y: scroll;
        }
        .tiempo-dias-siguientes {
            width: 100%;
            min-height: 30%;
            border-bottom-left-radius: 5px;
            border-bottom-right-radius: 5px;
            border-top-right-radius: 0px;
            flex-direction: row;
        }
        .tiempo-dias {
            flex-direction: column;
            max-height: 60px;
        }
    }
    
    @media (max-height: 700px) and (orientation: landscape) {
        .contenedor {
            overflow-y: auto;
            height: auto;
        }
        .tiempo-hoy {
            height: 500px;
            font-size: 14px;
        }
        .tiempo-dias {
            height: auto;
        }
        #temperatura-valor {
            font-size: 12px;
        }
    }
    
    @media (max-height: 700px) {
        #temperatura-valor {
            font-size: 12px;
        }
    }
    
    @media (max-width: 550px) {
        .contenedor{
            padding: 50px;
        }
        .tiempo-dias-siguientes {
            flex-wrap: wrap;
            overflow-y: auto;
        }
        .tiempo-hoy {
            min-height: 80%;
            overflow-x: auto;
        }
        #icono-animado{
            width: 100px;
            height: 100px;
        }
    }

    @media (max-width:400px) {
        .contenedor{
            padding: 40px;
        }

        .input-buscador{
            width: 70%;
        }
    }
</style>