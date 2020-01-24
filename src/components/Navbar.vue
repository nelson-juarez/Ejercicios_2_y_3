<template>
	<nav>
		<v-app-bar app class="grey lighten-5" dark >
			<v-toolbar-title class="text-uppercase grey--text">
				<span class="font-weight-bold"> Codeicus </span>
				<span class="font-weight-light">- Ejercicios 2 y 3</span>
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
										item-text="name"
										item-value="cod"
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
													<v-tooltip bottom>
														<template v-slot:activator="{ on }">
															<v-btn text v-on="on" icon color="red" @click="quitarProducto(item.cod)">
																<v-icon>delete</v-icon>
															</v-btn>
														</template>
														<span>Quitar producto</span>
													</v-tooltip>
												</td>
												<td>{{ item.cod }}</td>
												<td>{{ item.name }}</td>
												<td>
													<v-chip v-if="item.stock==0" class="ma-2" color="red" dark >
														Sin stock
													</v-chip>
													<span v-if="item.stock"> {{ item.stock }} </span>
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
											></v-text-field>
										</v-flex>
										<v-flex sm4 xs12 class="pa-sm-2 ">
											<v-text-field
												v-model.number="v_nombre"
												label="Nombre Receptor"
												:required="required"
												:rules="[rulesEnvio]"
											></v-text-field>
										</v-flex>
										<v-flex sm4 xs12 class="pa-sm-2 ">
											<v-menu
												v-model="menu"
												:close-on-content-click="false"
												max-width="290"
												>
												<template v-slot:activator="{ on }">
													<v-text-field
													:value="v_fechaEntrega"
													clearable
													label="Fecha de entrega"
													readonly
													v-on="on"
													prepend-icon="event"
													@click:clear="v_fechaEntrega = null"
													></v-text-field>
												</template>
												<v-date-picker
												:min="nowDate"
													v-model="v_fechaEntrega"
													@change="menu = false"
												></v-date-picker>
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
import format from 'date-fns/format'
  export default {
	data: vm => ({
		v_producto: null,
		arProductos: [
					{ id:  1, "cod": 1,"name": "Producto 1", "stock": 0 },
					{ id:  2, "cod": 2,"name": "Producto 2", "stock": 20 },
					{ id:  3, "cod": 3,"name": "Producto 3", "stock": 7 },
					{ id:  4, "cod": 4,"name": "Producto 4", "stock": 2 },
					{ id:  5, "cod": 5,"name": "Producto 5", "stock": 0 },
					{ id:  6, "cod": 6,"name": "Producto 6", "stock": 9 },
					{ id:  7, "cod": 7,"name": "Producto 7", "stock": 21 },
					{ id:  8, "cod": 8,"name": "Producto 8", "stock": 31 },
					{ id:  9, "cod": 9,"name": "Producto 9", "stock": 7 },
					{ id:  10, "cod": 10,"name": "Producto 10", "stock": 10 },
					{ id:  11, "cod": 11,"name": "Producto 11", "stock": 1 },
					{ id:  12, "cod": 12,"name": "Producto 12", "stock": 0 },
					{ id:  13, "cod": 13,"name": "Producto 13", "stock": 15 },
					{ id:  14, "cod": 14,"name": "Producto 14", "stock": 13 },
					{ id:  15, "cod": 15,"name": "Producto 15", "stock": 11 },
					{ id:  16, "cod": 16,"name": "Producto 16", "stock": 12 },
					{ id:  17, "cod": 17,"name": "Producto 17", "stock": 7 },
					{ id:  18, "cod": 18,"name": "Producto 18", "stock": 0 },
					{ id:  19, "cod": 19,"name": "Producto 19", "stock": 9 },
					{ id:  20, "cod": 20,"name": "Producto 20", "stock": 23 }
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
		nowDate: new Date().toISOString().substr(0, 10),
		v_fechaEntrega: null,
		menu: false,
		loadingProc: false,
		procesoExitoso: false,
		procesoErroneo: false,
		procesoCompletado: null
	}),
	computed: {
		formattedDate () {
			console.log( this.v_fechaEntrega );
			return this.v_fechaEntrega ? format(this.v_fechaEntrega, 'Do MMM YYYY') : '';
		}
	},
	methods: {
	  validate () {
		if (this.$refs.form.validate()) {
			// Verifico que lo ingresado no supere lo disponible

			// Obtengo el stock disponible para el producto en cuestión. Por defecto es lo que viene, llega al obtener los datos.
			var cantiDisponible = this.v_producto.stock;

			// Si ya fue cargado debo obtener el disponible
			if( this.arStockDisponible.length > 0 )
				if( this.arStockDisponible.find(obj => obj.cod === this.v_producto.cod ))
					cantiDisponible = this.arStockDisponible.find(obj => obj.cod === this.v_producto.cod ).stock;
			console.log( cantiDisponible);
			// Si lo ingresado supero lo disponible muestro un mensaje de error
			if( this.v_cantidad > cantiDisponible ) {
				this.maxCantidad = cantiDisponible;
				this.errorProducto = true;
			} else {
				this.errorProducto = false;
				this.maxCantidad = null;
				var nuevo = false;

				if( this.arStockDisponible.findIndex(obj => obj.cod === this.v_producto.cod) >= 0 ) {        
					// Actualizo la cantidad disponible
					this.arStockDisponible.filter(obj => { 
						if(obj.cod === this.v_producto.cod){
							obj.stock -= this.v_cantidad;
						}
						return obj;
					});

					nuevo = true;
				} else {
					console.log('pasa 1' );
					console.log( this.v_producto.stock +'-'+ this.v_cantidad );
					// Si lo ingreso el stock disponible va a ser lo que tiene menos lo que se agregó
					this.arStockDisponible.push({ 
						cod: this.v_producto.cod,
						stock: this.v_producto.stock - this.v_cantidad
					});
					console.log( this.arStockDisponible );
				}

				if( !nuevo ) {
					console.log( this.v_producto );
					// Inserto el producto en la tabla
					this.arProdPedido.push({
						id: this.v_producto.id,
						cod: this.v_producto.cod,
						name: this.v_producto.name,
						stock: this.v_cantidad
					});
				} else {
					this.arProdPedido.filter(obj => { 
						if(obj.cod === this.v_producto.cod){
							obj.stock += this.v_cantidad;
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
		this.$refs.form.resetValidation();
		this.$refs.formOpc.resetValidation();
	  },
	  quitarProducto (codProducto){
		  this.arProdPedido = this.arProdPedido.filter( e => e.cod != codProducto);
		  // Actualizo el stock disponible para el producto en cuestión.
		  this.arStockDisponible = this.arStockDisponible.filter( e => e.cod != codProducto);
	  },
	  	validateEnvio (){
			// Antes de enviar el pedido verifico que el formulario sea válido
			var validEnvio = this.$refs.formOpc.validate();

			if( validEnvio ) {
				// Si el formulario es válido genero el proceso de envío y reseteo los valores
				if( this.arProdPedido.length > 0 && validEnvio) {
					this.loadingProc = true;
					setTimeout( () => {
						this.procesoCompletado=true;
						this.loadingProc = false; 
						this.procesoExitoso = true;
						this.arStockDisponible = [];
						this.arProdPedido = [];
						this.resetValidation();
						
						setTimeout( () => {
							this.procesoCompletado = null;
							this.procesoErroneo = this.procesoExitoso = false;
							this.e6 = 1;
						}, 3000);
					}, 3000);
				}
			}
	  	},
	  	rulesEnvio (v) {
			var valid = true;

			if (this.required) {
				return !!v || 'El campo es requerido';
			}

			return valid;
		  }
	},
	watch: {
	  v_producto (val) {
		if (val != null) this.tab = 0
		else this.tab = null
	  },
	  v_dni (val){
		  if (val || this.v_nombre ) this.required = true;
		  else this.required = false;
	  },
	  v_nombre (val){
		  if (val || this.v_dni) this.required = true;
		  else this.required = false;
	  }
	}
  }
</script>