<template>
  <v-container>
    <v-row no-gutters>
      <v-col md="8" offset-md="2">
        <v-card class="pa-2" outlined tile>
          <v-form ref="myForm">
          <v-col sm="6" class="py-2">Twój obecny rachunek za prąd <h3 class="py-5 text-center">{{ price }} zł</h3></v-col>
          <v-col sm="6" class="py-2">Twoje zapotrzebowanie na miesiąc: <h3 class="py-5 text-center">{{ countDemand }} kWh</h3></v-col>
          <v-slider max="2500" min="0" v-model="price" color="#D0794E" track-color="#000" thumb-color="#000" required></v-slider>
          <v-col class='d-flex justify-center'>
          <v-checkbox class="text-center py-5 font-weight-medium" label="Planuję magazyn energii" color="#000" v-model="battery" required @change="calculatePrice"></v-checkbox>
          </v-col>
          <v-col class='d-flex justify-center mb-10' v-if="battery">
              <v-radio-group v-model="batteryCapacity" @change="calculatePrice" row required>
                <v-radio label="Do 10 kWh" value="10" color="#D0794E"></v-radio>
                <v-radio label="Do 15 kWh" value="15" color="#D0794E"></v-radio>
                <v-radio label="Do 20 kWh" value="20" color="#D0794E"></v-radio>
            </v-radio-group>
          </v-col>
          <h3 class="text-center">Gdzie instalacja?</h3>
          <v-col class="d-flex justify-center">
          <v-select :items="whereItems" label="Typ instalacji:" color="#D0794E" item-color="#000" v-model="type" required @change="calculatePrice"></v-select>
          </v-col>
          <!-- Gruntowa -->
          <div v-if="type === 'Gruntowa'">
          <h3 class="text-center">Rodzaj instalacji gruntowej</h3>
          <v-col class="d-flex justify-center">
            <v-radio-group v-model="landTypeRadio" required @change="calculatePrice">
              <v-radio label="Statyczna" value="Statyczna" color="#D0794E"></v-radio>
              <v-radio label="Ruchoma" value="Ruchoma" color="#D0794E"></v-radio>
            </v-radio-group>
          </v-col>
          <div v-if="landTypeRadio === 'Ruchoma'">
           <v-col class="d-flex justify-center pb-10">
              <h3 class="text-center py-3 red--text">Zmień wyżej na tracker.</h3>
           </v-col>
          </div>
          <!-- END Ruchoma -->
          <h3 class="text-center">Typ:</h3>
          <v-col class="d-flex justify-center">
            <v-radio-group v-model="landRadio" required @change="calculatePrice">
              <v-radio label="Stelaż" value="Stelaż" color="#D0794E"></v-radio>
              <v-radio label="Carport" value="Carport" color="#D0794E"></v-radio>
            </v-radio-group>
          </v-col>
          <h3 class="text-center">Moc Twojej instalacji:</h3>
          <h2 class="d-flex justify-center py-10">{{ power }} kWp</h2>
           <v-col class="d-flex justify-center pb-10" v-if="landRadio === 'Carport'">
              <h3 class="text-center py-3 green--text">Wybierz moc carportu od 1 do 6 kW!</h3>
           </v-col>
          <v-col class="d-flex justify-center">
            <v-slider max="40" min="0" v-model="power" color="#D0794E" track-color="#000" thumb-color="#000" required @change="calculatePrice"></v-slider>
          </v-col>
          <!-- END Gruntowa -->
          </div>

          <div v-else-if="type === 'Dachowa'">
            <h3 class="text-center pt-10">Jaki masz dach?</h3>
            <v-col class="d-flex justify-center pb-10">
              <v-select :items="roofItems" label="Dach:" v-model="roofItem" color="#D0794E" item-color="#000" required @change="calculatePrice"></v-select>
            </v-col>
             <h3 class="text-center">Moc Twojej instalacji:</h3>
          <h2 class="d-flex justify-center py-10">{{ power }} kWp</h2>
          <v-col class="d-flex justify-center">
            <v-slider max="40" min="0" v-model="power" color="#D0794E" track-color="#000" thumb-color="#000" required @change="calculatePrice"></v-slider>
          </v-col>
          <!-- END Dachowa -->
          </div>
          <div v-else-if="type === 'Tracker'">
            Tracker
            <h3 class="text-center">Moc instalacji:</h3>
            <v-col class="d-flex justify-center py-10">
              <v-radio-group v-model="trackerRadio" row required @change="calculatePrice">
                <v-radio label="T8 - 4 kWp" value="T8" color="#D0794E"></v-radio>
                <v-radio label="T12 - 6,5 kWp" value="T12" color="#D0794E"></v-radio>
                <v-radio label="T15 - 8 kWp" value="T15" color="#D0794E"></v-radio>
              </v-radio-group>
            </v-col>
          <!-- END Tracker -->
          </div>
          <h3 class="text-center">Planowany miesiąc montażu:</h3>
          <v-date-picker v-model="date" full-width type="month" class="mt-4" locale="pl" color="#D0794E" required @change="calculatePrice"></v-date-picker>
          <h2 class="text-center py-10">Przybliżona cena: <span style="color:#D0794E; font-size: 1.2em;">{{ amount }} </span><span v-if="amount" style="font-size: 1.2em;">000 zł brutto</span></h2>
          <!-- Start Personal Data -->
          <v-col align-self="center" offset="2" cols="8" md="6" offset-md="3">
            <div v-if="awesome">
              <v-text-field label="Imię" color="#D0794E" v-model="name" required></v-text-field>
              <span v-if="msg.name" class="red--text pb-3">{{msg.name}}</span>
              <v-text-field label="Nazwisko" color="#D0794E" v-model="surname" required></v-text-field>
              <span v-if="msg.surname" class="red--text pb-3">{{msg.surname}}</span>
              <v-text-field label="Telefon" color="#D0794E" v-model="telephone" required></v-text-field>
              <span v-if="msg.telephone" class="red--text pb-3">{{msg.telephone}}</span>
              <v-text-field label="E-mail" color="#D0794E" v-model="email" required></v-text-field>
              <span v-if="msg.email" class="red--text pb-3">{{msg.email}}</span>
              <v-text-field label="Miejscowość" color="#D0794E" v-model="city" required></v-text-field>
              <span v-if="msg.city" class="red--text pb-3">{{msg.city}}</span>
              <v-text-field label="Kod pocztowy" color="#D0794E" v-model="code" required></v-text-field>
              <span v-if="msg.code" class="red--text pb-3">{{msg.code}}</span>
              <v-col class="d-flex justify-center pt-5" v-if="showFormError">
              <span class="red--text font-weight-bold">{{ errorMsg }}</span>
              </v-col>
            </div>
            <v-col class="d-flex justify-center pt-5" v-if="sent">
            <span class="green--text font-weight-bold">Formularz został wysłany! Skontaktujemy się z Tobą w ciągu 24h!</span>
            </v-col>
            <v-col class="d-flex justify-center py-10">
            <v-btn outlined color="#D0794E" @click="submitForm" x-large>Wyślij</v-btn>
            </v-col>
          </v-col>
          <!-- END Personal Data -->
          </v-form>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import mailjs from 'emailjs-com';
  export default {
    name: 'CalculatorApp',

    data: () => ({
      amount: 0,
      price: 0,
      power: 0,
      sent: false,
      landTypeRadio: null,
      landRadio: null,
      trackerRadio: null,
      batteryCapacity: null,
      type: null,
      battery: false,
      awesome: true,
      whereItems: ['Gruntowa', 'Dachowa', 'Tracker'],
      roofItems: ['Dachówka', 'Gont', 'Blacho-dachówka', 'Trapez', 'Blacha na rąbek', 'Płyta wielowarstwowa'],
      roofItem: null,
      column: null,
      row: null,
      date: (new Date(Date.now() - (new Date()).getTimezoneOffset() * 60000)).toISOString().substr(0, 10),
      name: null,
      surname: null,
      telephone: null,
      email: null,
      city: null,
      code: null,
      msg: {
        name: '',
        surname: '',
        telephone: '',
        email: '',
        city: '',
        code: ''
      },
      showFormError: false,
      errorMsg: ''
    }),
     computed: {
      countDemand: function() {
      return Math.round(this.price * 1.29)
      }
    },
    watch: {
      email(value){
      this.email = value;
      this.validateEmail(value);
    },
    name(value){
      this.name = value;
      this.validateName(value);
    },
    surname(value){
      this.surname = value;
      this.validateSurname(value);
    },
    telephone(value){
      this.telephone = value;
      this.validateTelephone(value);
    },
    city(value){
      this.city = value;
      this.validateCity(value);
    },
    code(value){
      this.code = value;
      this.validateCode(value);
    }
  },
    methods : {
      // email
      validateEmail(value){
      if (/[a-zA-Z0-9.]*@[a-z]*[.a-z]*/.test(value))
            {
              this.msg.email = '';
            } else{
              this.msg.email = 'Niepoprawny adres e-mail!';
            }
        },
        validateName(value){
          if(/[a-zA-Z]{3,30}/.test(value)){
            this.msg.name = '';
          }else{
            this.msg.name = 'Niepoprawne imię!';
          }
        },
        validateSurname(value){
            if(/[a-zA-Z]{3,30}/.test(value)){
              this.msg.surname = '';
            }else{
              this.msg.surname = 'Niepoprawne nazwisko!';
            }
        },
        // telefon
        validateTelephone(value){
          if(/(?<!\w)(\(?(\+|00)?48\)?)?[ -]?\d{3}[ -]?\d{3}[ -]?\d{3}(?!\w)$/.test(value)){
            this.msg.telephone = '';
          }else{
            this.msg.telephone = 'Niepoprawny format numeru';
          }
        },
        validateCity(value){
          if(/^[\s\p{L}]+$/u.test(value)){
              this.msg.city = '';
            }else if( !/^[\s\p{L}]+$/u.test(value) ){
              this.msg.city = 'Dziwna nazwa miejscowości?';
            }
        },
        // kod pocztowy
        validateCode(value){
          if(/^[0-9]{2}-[0-9]{3}/.test(value)){
            this.msg.code = '';
          }else{
            this.msg.code = 'Niepoprawny format kodu pocztowego!';
          }
        },
        addBattery: function(){
          if( this.batteryCapacity == 10 ){
            this.amount = 0;
            this.amount = this.amount + 20;
          }
          
          if( this.batteryCapacity == 15 ){
            this.amount = 0;
            this.amount = this.amount + 25;
          }

          if( this.batteryCapacity == 20 ){
            this.amount = 0;
            this.amount = this.amount + 30;
          }
        },
        calculatePrice: function(){
          this.amount = 0;
          // #1 calc Gruntowa
          // Do 5kw: 1kwp - 6k Powyzej 5kw: 1kwp - 5k

          if( this.battery ){
            this.addBattery();
          }
          
          if(this.type === 'Gruntowa'){
            // 1kw - 5.5k
            // Gruntowa statyczna stelaż
            if( this.power <= 6 && this.power >= 1 ){
              this.amount = this.amount + (this.power *6);
              console.log('Calculate 1 - 5 kw');
            } else if( this.power >= 7 && this.power <= 40 ){
              this.amount = this.amount + (this.power * 4);
            }
            // Gruntowa statyczna carport
            if(this.landRadio === 'Carport'){
              if( this.power <= 6 && this.power >=1 ){
                this.amount = 0;
                this.amount = this.amount + Math.round(this.power * 6.7);
              }else{
                this.amount = 0;
              }
            }
            
          }

          // #2 calc Dachowa

          if(this.type === 'Dachowa'){
            //Do 5kw: 1kwp - 5k Powyżej 6kw: 1kwp - 4.5k
            if( this.roofItem === 'Dachówka'){
              this.amount = this.amount + (this.power * 0.5);
            }
            if( this.roofItem === 'Gont' || this.roofItem === 'Płyta wielowarstwowa'){
              this.amount = this.amount + (this.power * 0.2);
            }
            if( this.roofItem === 'Blacho-dachówka' || this.roofItem === 'Blacha na rąbek'){
              this.amount = this.amount + (this.power * 0.4);
            }
            if( this.roofItem === 'Trapez' ){
              this.amount = this.amount + (this.power * 0.2);
            }
            if( this.power <= 6 && this.power >= 1 ){
              this.amount = this.amount +  Math.round(this.power * 5.8);
            } else if( this.power >= 7 && this.power <= 40 ){
              this.amount = this.amount + Math.round( this.power * 4.3 );
            }
          }

          // #3 calc Tracker

          if(this.type === 'Tracker'){
            if( this.trackerRadio === 'T8' )
            {
              this.amount = this.amount + 34;
              this.power = 5;
            }
            if( this.trackerRadio === 'T12' )
            {
              this.amount = this.amount + 39;
              this.power = 6;
            }
            if( this.trackerRadio === 'T15' )
            {
              this.amount = this.amount + 48;
              this.power = 8;
            }
          }

          return this.amount;
        },
        submitForm: function(){
          if( this.amount && this.name != null 
              && this.surname != null 
              && this.telephone != null 
              && this.email != null 
              && this.city != null 
              && this.code != null)
            {
              this.showFormError = false;
              let templateParams = {
                  name: this.name,
                  surname: this.surname,
                  telephone: this.telephone,
                  email: this.email,
                  city: this.city,
                  code: this.code,
                  //instalacja
                  power: this.power,
                  price: this.price,
                  countDemand: this.countDemand,
                  batteryCapacity: this.batteryCapacity,
                  type: this.type,
                  landTypeRadio: this.landTypeRadio,
                  landRadio: this.landRadio,
                  roofItem: this.roofItem,
                  trackerRadio: this.trackerRadio,
                  date: this.date,
                  amount: this.amount
                };
              mailjs.send("service_j43t1cn","template_667bgss", templateParams, 'F_pcL1FNA60JIt7Ej').then(function(response) {
                console.log('SUCCESS!', response.status, response.text);
              }, function(error) {
                console.log('FAILED...', error);
              });
              this.sent = true;
              this.awesome = false;
              this.$refs.myForm.reset();
            } else if( this.amount === 0 )
            {
              this.showFormError = true;
              this.errorMsg = 'Zrób wycenę swojej instalacji'
            }
              else{
              this.showFormError = true;
              this.errorMsg = 'Wypełnij poprawnie formularz!'
            }
        }
    }
}
</script>
<style>
.v-input .v-label{
  font-size: 1.3em;
  color: #000;
}
</style>
