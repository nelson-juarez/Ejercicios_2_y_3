<template>
	<nav>
		<v-app-bar app class="grey lighten-5" dark >
			<v-toolbar-title class="text-uppercase grey--text">
				<span class="font-weight-bold"> Codeicus </span>
				<span class="font-weight-light">- Ejercicio 1</span>
			</v-toolbar-title>
		</v-app-bar>
		<v-divider></v-divider>
		<v-container>
			<v-stepper v-model="e6" vertical class="white">
				<v-stepper-step :complete="e6 > 1" step="1" editable>
					Datos del pedido
				</v-stepper-step>

				<v-stepper-content step="1" >
					<div class="grey lighten-4 py-3">
						<v-layout wrap justify-center >
							<v-flex md6 sm12 class="px-5">
								<h1 class="text-center mb-5">Cargar producto</h1>
								<v-card color="white elevation-5" class="mb-12">
									<v-form ref="form" v-model="valid" lazy-validation class="pa-4">
										<v-select
										v-model="v_producto"
										:items="arProductos"
										item-text="nombre"
										item-value="codigo"
										:rules="[v => !!v || '* El campo es requerido']"
										label="Producto"
										required
										prepend-icon="add_shopping_cart"
										single-line
										return-object
										>
										</v-select>
										<v-text-field
											v-model.number="v_cantidad"
											type="number"
											label="Cantidad"
											prepend-icon="list"
											:rules="[v => !!v || '* El campo es requerido', v => (v && v <= 500) || 'No se puede ingresar una cantidad superior a 500']"
											required
										></v-text-field>
										<v-btn color="primary" block  class="mt-5 my-2" outlined :disabled="!valid" @click="validate">Agregar</v-btn>
										<v-alert
											text
											prominent
											type="error"
											border="left"
											dismissible
											outlined
											class="my-3"
											v-if="errorProducto"
											>
											<v-row align="center">
												<v-col class="grow"> 
													<span v-if="this.maxCantidad > 0 ">
														La cantidad ingresada supera la cantidad disponible para el producto seleccionado. <br> 
														El stock disponible es: <strong> {{ this.maxCantidad }} </strong> 
													</span>
													<span v-else>
														El producto no cuenta con stock disponible
													</span>
												</v-col>
											</v-row>
										</v-alert>
										<v-spacer></v-spacer>
									</v-form>
								</v-card>
							</v-flex>
							<v-flex md6 sm12 class="px-5 mb-12">
								<h1 class="text-center mb-5">Productos a despachar</h1>
								<v-simple-table v-if="arProdPedido.length > 0">
									<template v-slot:default>
										<thead>
											<tr>
												<th class="text-center">-</th>
												<th class="text-left">Código</th>
												<th class="text-left">Nombre</th>
												<th class="text-left">Cantidad</th>
											</tr>
										</thead>
										<tbody>
											<tr v-for="item in arProdPedido" :key="item.codigo">
												<td class="text-center"> 
													<v-btn text icon color="red" @click="quitarProducto(item.codigo)">
														<v-icon>delete</v-icon>
													</v-btn>
												</td>
												<td>{{ item.codigo }}</td>
												<td>{{ item.nombre }}</td>
												<td>
													<v-chip v-if="item.cantidad==0" class="ma-2" color="red" dark >
														Sin stock
													</v-chip>
													<span v-if="item.cantidad"> {{ item.cantidad }} </span>
												</td>
											</tr>
										</tbody>                                
									</template>
								</v-simple-table>

								<v-alert v-else type="error" class="mt-3"> No hay productos cargados </v-alert>
								
							</v-flex>
						</v-layout>

						<v-spacer></v-spacer>

						<div class="text-center">
							<v-btn color="primary" :disabled="arProdPedido.length==0 " @click="validarProductos">Guardar</v-btn>
						</div>
					</div>
				</v-stepper-content>

				<v-stepper-step :complete="e6 > 2" step="2">
					Datos del envío
					<small class="mt-1">(Opcionales)</small>
				</v-stepper-step>

				<v-stepper-content step="2">
					<div class="grey lighten-4 py-3 px-5">
						<h1 class="text-center mb-5">Ingrese los datos de envío</h1>
						<v-card color="white elevation-5" class="mb-12" >
							<v-form
								ref="formOpc"
								v-model="validOpc"
								lazy-validation
								class="pa-4">
									<v-layout wrap>
										<v-flex sm4 xs12 class="pa-sm-2 " >
											<v-text-field
												v-model.number="v_dni"
												label="DNI Receptor"
												:required="required"
												:rules="[rulesEnvio]"
												@blur="rulesEnvio"
											></v-text-field>
										</v-flex>
										<v-flex sm4 xs12 class="pa-sm-2 ">
											<v-text-field
												v-model.number="v_nombre"
												label="Nombre Receptor"
												:required="required"
												:rules="[rulesEnvio]"
												@blur="rulesEnvio"
											></v-text-field>
										</v-flex>
										<v-flex sm4 xs12 class="pa-sm-2 ">
											<v-menu
												ref="menu"
												v-model="menu"
												:close-on-content-click="false"
												:return-value.sync="v_fechaEntrega"
												transition="scale-transition"
												offset-y
												min-width="290px"
											>
												<template v-slot:activator="{ on }">
												<v-text-field
													v-model="dateFormatted"
													label="Fecha de entrega"
													prepend-icon="event"
													readonly
													@blur="v_fechaEntrega = parseDate(dateFormatted)"
													v-on="on"
												></v-text-field>
												</template>
												<v-date-picker v-model="v_fechaEntrega" no-title scrollable :min="nowDate" @input="menu = false" />
											</v-menu>                        
										</v-flex>
									</v-layout>
									<v-layout>
										<v-flex sm12>
											<v-textarea
												outlined
												v-model = "v_comentario"
												label="Comentarios"
												counter
												:rules=" [v => v.length <= 100 || 'Máximo 100 caracteres']"
												placeholder="Ingrese comentarios u observaciones que considere importante"
												></v-textarea>
										</v-flex>
									</v-layout>
							</v-form
							>
						</v-card>

						<v-alert
							text
							prominent
							type="success"
							border="left"
							dismissible
							outlined
							class="my-3"
							v-if="procesoCompletado"
							>
							<v-row align="center">
								<v-col class="grow"> 
									<span v-if="procesoExitoso">
										El proceso fue realizado correctamente.
									</span>
									<span v-if="procesoErroneo">
										Se produjo un error al enviar el pedido. Inténtelo nuevamente más tarde.
									</span>
								</v-col>
							</v-row>
						</v-alert>
						
						<div class="text-center">
							<v-btn  class="ma-2" color="primary" :disabled="!validOpc && loadingProc" 
									@click="validateEnvio" :loading="loadingProc">
								Procesar envío
							</v-btn>
						</div>
					</div>
				</v-stepper-content>
			</v-stepper>
		</v-container>
	</nav>
</template>

<script>

  export default {
	data: vm => ({
		v_producto: null,
		arProductos: [
					{ "codigo": 1,"nombre": "Producto 1", "cantidad": 0 },
					{ "codigo": 2,"nombre": "Producto 2", "cantidad": 20 },
					{ "codigo": 3,"nombre": "Producto 3", "cantidad": 7 },
					{ "codigo": 4,"nombre": "Producto 4", "cantidad": 2 },
					{ "codigo": 5,"nombre": "Producto 5", "cantidad": 0 },
					{ "codigo": 6,"nombre": "Producto 6", "cantidad": 9 },
					{ "codigo": 7,"nombre": "Producto 7", "cantidad": 21 },
					{ "codigo": 8,"nombre": "Producto 8", "cantidad": 31 },
					{ "codigo": 9,"nombre": "Producto 9", "cantidad": 7 },
					{ "codigo": 10,"nombre": "Producto 10", "cantidad": 10 },
					{ "codigo": 11,"nombre": "Producto 11", "cantidad": 1 },
					{ "codigo": 12,"nombre": "Producto 12", "cantidad": 0 },
					{ "codigo": 13,"nombre": "Producto 13", "cantidad": 15 },
					{ "codigo": 14,"nombre": "Producto 14", "cantidad": 13 },
					{ "codigo": 15,"nombre": "Producto 15", "cantidad": 11 },
					{ "codigo": 16,"nombre": "Producto 16", "cantidad": 12 },
					{ "codigo": 17,"nombre": "Producto 17", "cantidad": 7 },
					{ "codigo": 18,"nombre": "Producto 18", "cantidad": 0 },
					{ "codigo": 19,"nombre": "Producto 19", "cantidad": 9 },
					{ "codigo": 20,"nombre": "Producto 20", "cantidad": 23 }
				],
		v_cantidad: null,
		e6: 1,
		valid: false,
		arProdPedido: [],
		arStockDisponible: [],
		errorProducto: false,
		maxCantidad: null,
		required: false,
		v_dni: null,
		v_nombre: null,
		v_comentario: '',
		validOpc: true,
		nowDate: new Date().toISOString().slice(0,10),
		dateFormatted: vm.formatDate(new Date().toISOString().substr(0, 10)),
		v_fechaEntrega: new Date().toISOString().substr(0, 10),
		menu: false,
		loadingProc: false,
		procesoExitoso: false,
		procesoErroneo: false,
		procesoCompletado: null
	}),
	computed: {
	  	computedDateFormatted () {
			return this.formatDate(this.v_fechaEntrega)
	  	}
	},
	methods: {
	  validate () {
		if (this.$refs.form.validate()) {
			// Verifico que lo ingresado no supere lo disponible

			// Obtengo el stock disponible para el producto en cuestión. Por defecto es lo que viene, llega al obtener los datos.
			var cantiDisponible = this.v_producto.cantidad;

			// Si ya fue cargado debo obtener el disponible
			if( this.arStockDisponible.length > 0 )
				if( this.arStockDisponible.find(obj => obj.codigo === this.v_producto.codigo ))
					cantiDisponible = this.arStockDisponible.find(obj => obj.codigo === this.v_producto.codigo ).cantidad;
			
			// Si lo ingresado supero lo disponible muestro un mensaje de error
			if( this.v_cantidad > cantiDisponible ) {
				this.maxCantidad = cantiDisponible;
				this.errorProducto = true;
			} else {
				this.errorProducto = false;
				this.maxCantidad = null;
				var nuevo = false;

				if( this.arStockDisponible.findIndex(obj => obj.codigo === this.v_producto.codigo) >= 0 ) {        
					// Actualizo la cantidad disponible
					this.arStockDisponible.filter(obj => { 
						if(obj.codigo === this.v_producto.codigo){
							obj.cantidad -= this.v_cantidad;
						}
						return obj;
					});

					nuevo = true;
				} else {
					// Si lo ingreso el stock disponible va a ser lo que tiene menos lo que se agregó
					this.arStockDisponible.push({ 
						codigo: this.v_producto.codigo,
						cantidad: this.v_producto.cantidad - this.v_cantidad
					});
				}

				if( !nuevo ) {
					// Inserto el producto en la tabla
					this.arProdPedido.push({
						codigo: this.v_producto.codigo,
						nombre: this.v_producto.nombre,
						cantidad: this.v_cantidad
					});
				} else {
					this.arProdPedido.filter(obj => { 
						if(obj.codigo === this.v_producto.codigo){
							obj.cantidad += this.v_cantidad;
						}
						return obj;
					});
				}
			}

		  this.v_producto=null;
		  this.v_cantidad=null;
		  this.resetValidation();
		}
	  },
	  validarProductos () {
		  	if( this.arProdPedido.length > 0){
				this.e6 = 2
				this.maxCantidad = 0;
				this.errorProducto = false;
			}
	  },
	  resetValidation () {
		this.$refs.form.resetValidation()
	  },
	  quitarProducto (codProducto){
		  this.arProdPedido = this.arProdPedido.filter( e => e.codigo != codProducto);
		  // Actualizo el stock disponible para el producto en cuestión.
		  this.arStockDisponible = this.arStockDisponible.filter( e => e.codigo != codProducto);
	  },
	  	validateEnvio (){
			var validEnvio = (this.required && ( this.v_dni && this.v_nombre) ) ||
							 (!this.required && ( !this.v_dni && !this.v_nombre) );
			if( this.arStockDisponible.length > 0 && validEnvio) {
				this.loadingProc = true;
				setTimeout( () => {
					this.procesoCompletado=true;
					this.loadingProc = false; 
					this.procesoExitoso = true;
					this.arStockDisponible = [];
					this.arProdPedido = [];
					this.v_dni = this.v_nombre = this.v_comentario = this.v_producto = this.v_cantidad = null;
					this.v_fechaEntrega = new Date().toISOString().substr(0, 10);

					setTimeout( () => {
						this.procesoCompletado = null;
						this.procesoErroneo = this.procesoExitoso = false;
						this.e6 = 1;
					}, 3000);
				}, 3000);
			}

	  	},
	  	rulesEnvio (value) {
			var valid = true;

			if (this.required && value.length === 0) {
				valid = "El campo es requerido.";  
			}

			return valid;
		},
	formatDate (date) {
		if (!date) return null

		const [year, month, day] = date.split('-')
		return `${day}/${month}/${year}`
	  },
	  parseDate (date) {
		if (!date) return null

		const [month, day, year] = date.split('/')
		return `${year}-${month.padStart(2, '0')}-${day.padStart(2, '0')}`
	  }
	
	},
	watch: {
	  v_producto (val) {
		if (val != null) this.tab = 0
		else this.tab = null
	  },
	  search (val) {
		// Items have already been loaded
		if (this.items.length > 0) return

		fetch('API/productos.json')
		  .then(res => res.json())
		  .then(res => {
			this.items = res.data
		  })
		  .catch(err => {
			console.log(err)
		  })
		  .finally(() => (this.isLoading = false))
	  },
	  v_dni (val){
		  if (val || this.v_nombre ) this.required = true;
		  else this.required = false;
	  },
	  v_nombre (val){
		  if (val || this.v_dni) this.required = true;
		  else this.required = false;
	  },
	  v_fechaEntrega (val) {
		this.dateFormatted = this.formatDate(this.v_fechaEntrega)
	  }
	}
  }
</script>


<style>
  .custom-loader {
    animation: loader 1s infinite;
    display: flex;
  }
  @-moz-keyframes loader {
    from {
      transform: rotate(0);
    }
    to {
      transform: rotate(360deg);
    }
  }
  @-webkit-keyframes loader {
    from {
      transform: rotate(0);
    }
    to {
      transform: rotate(360deg);
    }
  }
  @-o-keyframes loader {
    from {
      transform: rotate(0);
    }
    to {
      transform: rotate(360deg);
    }
  }
  @keyframes loader {
    from {
      transform: rotate(0);
    }
    to {
      transform: rotate(360deg);
    }
  }
</style>