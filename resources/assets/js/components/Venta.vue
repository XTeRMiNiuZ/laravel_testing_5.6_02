<template>
<main class="main">
	<!-- Breadcrumb -->
	<ol class="breadcrumb">
		<li class="breadcrumb-item"><a href="/">Escritorio</a></li>
	</ol>
	<div class="container-fluid">
		<!-- Ejemplo de tabla Listado -->
		<div class="card">
			<div class="card-header">
				<i class="fa fa-align-justify"></i> Ventas
				<button type="button" @click="mostrarDetalle()" class="btn btn-secondary">
          <i class="icon-plus"></i>&nbsp;Nuevo
        </button>
			</div>
      <!-- listado -->
      <template v-if="listado == 1">
        <div class="card-body">
  				<div class="form-group row">
  					<div class="col-12 col-sm-12 col-md-6">
  						<div class="input-group">
  							<select class="form-control col-12 col-sm-12 col-md-3" v-model="criterio">
                  <option value="tipo_comprobante">Tipo Comprobante</option>
                  <option value="num_comprobante">Número Comprobante</option>
                  <option value="fecha_hora">Fecha-Hora</option>
                </select>
  							<input type="text" v-model="buscar" @keyup.enter="listarVenta(1,buscar,criterio)" class="form-control" placeholder="Texto a buscar">
  							<button type="submit" @click="listarVenta(1,buscar,criterio)" class="btn btn-primary"><i class="fa fa-search"></i> Buscar</button>
  						</div>
  					</div>
  				</div>
  				<div class="table-responsive">
  					<table class="table table-bordered table-striped table-sm">
  						<thead>
  							<tr>
  								<th>Opciones</th>
  								<th>Usuario</th>
  								<th>Cliente</th>
  								<th>Tipo Comprobante</th>
  								<th>Serie Comprobante</th>
  								<th>Número Comprobante</th>
  								<th>Fecha Hora</th>
  								<th>Total</th>
  								<th>Impuesto</th>
  								<th>Estado</th>
  							</tr>
  						</thead>
  						<tbody>
  							<tr v-for="venta in arrayVenta" :key="venta.id">
  								<td>
  									<button type="button" @click="verVenta(venta.id)" class="btn btn-success btn-sm">
                      <i class="icon-eye"></i>
                    </button> &nbsp;
                    <button type="button" @click="pdfVenta(venta.id)" class="btn btn-info btn-sm">
                      <i class="icon-doc"></i>
                    </button> &nbsp;
  									<template v-if="venta.estado=='Registrado'">
                      <button type="button" class="btn btn-danger btn-sm" @click="desactivarVenta(venta.id)">
                        <i class="icon-trash"></i>
                      </button>
                    </template>
  								</td>
  								<td v-text="venta.usuario"></td>
  								<td v-text="venta.nombre"></td>
  								<td v-text="venta.tipo_comprobante"></td>
  								<td v-text="venta.serie_comprobante"></td>
  								<td v-text="venta.num_comprobante"></td>
  								<td v-text="venta.fecha_hora"></td>
  								<td v-text="venta.total"></td>
  								<td v-text="venta.impuesto"></td>
  								<td v-text="venta.estado"></td>
  							</tr>
  						</tbody>
  					</table>
  				</div>
  				<nav>
  					<ul class="pagination">
  						<li class="page-item" v-if="pagination.current_page > 1">
  							<a class="page-link" href="#" @click.prevent="cambiarPagina(pagination.current_page - 1,buscar,criterio)">Ant</a>
  						</li>
  						<li class="page-item" v-for="page in pagesNumber" :key="page" :class="[page == isActived ? 'active' : '']">
  							<a class="page-link" href="#" @click.prevent="cambiarPagina(page,buscar,criterio)" v-text="page"></a>
  						</li>
  						<li class="page-item" v-if="pagination.current_page < pagination.last_page">
  							<a class="page-link" href="#" @click.prevent="cambiarPagina(pagination.current_page + 1,buscar,criterio)">Sig</a>
  						</li>
  					</ul>
  				</nav>
  			</div>
      </template>
      <!-- fin Listado -->
      <!-- Detalle -->
      <template v-else-if="listado == 0">
        <div class="card-body">
          <div class="form-group row border">
            <div class="col-12 col-sm-12 col-md-9">
              <div class="form-group">
                <label for="">Cliente(*)</label>
                <v-select :on-search="selectCliente" label="nombre" :options="arrayCliente" placeholder="Buscar Clientes.." :onChange="getDatosCliente"></v-select>
              </div>
            </div>
            <div class="col-12 col-sm-12 col-md-3">
              <label for="">Impuesto(*)</label>
              <input type="text" class="form-control" v-model="impuesto" />
            </div>
            <div class="col-12 col-sm-12 col-md-4">
              <div class="form-group">
                <label for="">Tipo Comprobante(*)</label>
                <select class="form-control" v-model="tipo_comprobante">
                  <option value="0">Seleccione</option>
                  <option value="BOLETA">Boleta</option>
                  <option value="FACTURA">Factura</option>
                  <option value="TICKET">Ticket</option>
                </select>
              </div>
            </div>
            <div class="col-12 col-sm-12 col-md-4">
              <div class="form-group">
                <label for="">Serie Comprobante</label>
                <input type="text" class="form-control" v-model="serie_comprobante" placeholder="000x" />
              </div>
            </div>
            <div class="col-12 col-sm-12 col-md-4">
              <div class="form-group">
                <label for="">Número Comprobante(*)</label>
                <input type="text" class="form-control" v-model="num_comprobante" placeholder="000xx" />
              </div>
            </div>
            <div class="col-12">
              <div v-show="errorVenta" class="form-group row">
                <div class="col-12 text-center">
                  <div class="alert alert-danger">
                    <p class="mb-0" v-for="error in errorsMsgs" v-text="error"></p>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="form-group row border">
            <div class="col-12 col-sm-12 col-md-4">
              <div class="form-group">
                <label for="">Artículo <span style="color: red;" v-show="idarticulo==0">(*Seleccione)</span></label>
                <div class="form-inline">
                  <input type="text" class="form-control" v-model="codigo" @keyup.enter="buscarArticulo" placeholder="Ingrese artículo" />
                  <button class="btn btn-primary" @click="abrirModal">...</button>
                  <input type="text" readonly class="form-control" v-model="articulo" />
                </div>
              </div>
            </div>

            <div class="col-12 col-sm-12 col-md-2">
              <div class="form-group">
                <label for="">Precio <span style="color: red;" v-show="precio==0">(*Ingrese precio)</span></label>
                <input type="number" value="0" step="any" class="form-control" v-model="precio" />
              </div>
            </div>

            <div class="col-12 col-sm-12 col-md-2">
              <div class="form-group">
                <label for="">Cantidad <span style="color: red;" v-show="cantidad==0">(*Ingrese)</span></label>
                <input type="number" value="0" class="form-control" v-model="cantidad" />
              </div>
            </div>

            <div class="col-12 col-sm-12 col-md-2">
              <div class="form-group">
                <label for="">Descuento</label>
                <input type="number" value="0" class="form-control" v-model="descuento" />
              </div>
            </div>

            <div class="col-12 col-sm-12 col-md-2">
              <div class="form-group">
                <button class="btn btn-success form-control btn-add" @click="agregarDetalle"><i class="icon-plus"></i></button>
              </div>
            </div>
          </div>
          <div class="form-group row border">
            <div class="table-responsive col-12">
              <table class="table table-bordered table-striped table-sm">
                <thead>
                  <tr>
                    <th>Opciones</th>
                    <th>Artículo</th>
                    <th>Precio</th>
                    <th>Cantidad</th>
                    <th>Descuento</th>
                    <th>Subtotal</th>
                  </tr>
                </thead>
                <tbody v-if="arrayDetalle.length">
                  <tr v-for="(detalle, index) in arrayDetalle" :key="detalle.id">
                    <td>
                      <button type="button" class="btn btn-danger btn-sm" @click="eliminarDetalle(index)"><i class="icon-close"></i></button>
                    </td>
                    <td>
                      {{ detalle.articulo }}
                    </td>
                    <td>
                      <input v-model="detalle.precio" type="number" class="form-control" />
                    </td>
                    <td>
                      <span style="color: red;" v-show="detalle.cantidad > detalle.stock">Stock: {{ detalle.stock }}</span>
                      <input v-model="detalle.cantidad" type="number" class="form-control" />
                    </td>
                    <td>
                      <span style="color: red;" v-show="detalle.descuento > (detalle.precio*detalle.cantidad)">Descuento superior</span>
                      <input v-model="detalle.descuento" type="number" class="form-control" />
                    </td>
                    <td>
                      {{ detalle.precio*detalle.cantidad-detalle.descuento }}
                    </td>
                  </tr>
                  <tr style="background-color: #CEECF5;">
                    <td colspan="5" align="right"><strong>Total Parcial:</strong></td>
                    <td>{{ totalParcial = (total-totalImpuesto).toFixed(2) }}</td>
                  </tr>
                  <tr style="background-color: #CEECF5;">
                    <td colspan="5" align="right"><strong>Total Impuesto:</strong></td>
                    <td>{{ totalImpuesto = ((total*impuesto)/(1+impuesto)).toFixed(2) }}</td>
                  </tr>
                  <tr style="background-color: #CEECF5;">
                    <td colspan="5" align="right"><strong>Total Neto:</strong></td>
                    <td>{{ total = calcularTotal }}</td>
                  </tr>
                </tbody>
                <tbody v-else>
                  <tr>
                    <td colspan="6">
                      No hay artículos agregados
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
          <div class="form-group row">
            <div class="col-12">
              <button type="button" class="btn btn-secondary" @click="listado = 1">Cerrar</button>
              <button type="button" class="btn btn-primary" @click="registrarVenta()">Registrar Venta</button>
            </div>
          </div>
        </div>
      </template>
      <!-- Fin Detalle -->
      <!-- Ver ingreso Comentario -->
      <template v-else-if="listado == 2">
        <div class="card-body">
          <div class="form-group row border">
            <div class="col-12 col-sm-12 col-md-9">
              <div class="form-group">
                <label for="">Cliente</label>
                <p class="form-control-plaintext">{{ cliente }}</p>
              </div>
            </div>
            <div class="col-12 col-sm-12 col-md-3">
              <label for="">Impuesto</label>
              <p class="form-control-plaintext">{{ impuesto }}</p>
            </div>
            <div class="col-12 col-sm-12 col-md-4">
              <div class="form-group">
                <label for="">Tipo Comprobante</label>
                <p class="form-control-plaintext">{{ tipo_comprobante }}</p>
              </div>
            </div>
            <div class="col-12 col-sm-12 col-md-4">
              <div class="form-group">
                <label for="">Serie Comprobante</label>
                <p class="form-control-plaintext">{{ serie_comprobante }}</p>
              </div>
            </div>
            <div class="col-12 col-sm-12 col-md-4">
              <div class="form-group">
                <label for="">Número Comprobante</label>
                <p class="form-control-plaintext">{{ num_comprobante }}</p>
              </div>
            </div>
          </div>
          <div class="form-group row border">
            <div class="table-responsive col-12">
              <table class="table table-bordered table-striped table-sm">
                <thead>
                  <tr>
                    <th>Artículo</th>
                    <th>Precio</th>
                    <th>Cantidad</th>
                    <th>Descuento</th>
                    <th>Subtotal</th>
                  </tr>
                </thead>
                <tbody v-if="arrayDetalle.length">
                  <tr v-for="detalle in arrayDetalle" :key="detalle.id">
                    <td>
                      {{ detalle.articulo }}
                    </td>
                    <td>
                      {{ detalle.precio }}
                    </td>
                    <td>
                      {{ detalle.cantidad }}
                    </td>
                    <td>
                      {{ detalle.descuento }}
                    </td>
                    <td>
                      {{ detalle.precio*detalle.cantidad - detalle.descuento }}
                    </td>
                  </tr>
                  <tr style="background-color: #CEECF5;">
                    <td colspan="4" align="right"><strong>Total Parcial:</strong></td>
                    <td>{{ totalParcial = (total-totalImpuesto).toFixed(2) }}</td>
                  </tr>
                  <tr style="background-color: #CEECF5;">
                    <td colspan="4" align="right"><strong>Total Impuesto:</strong></td>
                    <td>{{ totalImpuesto = ((total*impuesto)).toFixed(2) }}</td>
                  </tr>
                  <tr style="background-color: #CEECF5;">
                    <td colspan="4" align="right"><strong>Total Neto:</strong></td>
                    <td>{{ total }}</td>
                  </tr>
                </tbody>
                <tbody v-else>
                  <tr>
                    <td colspan="5">
                      No hay artículos agregados
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
          <div class="form-group row">
            <div class="col-12">
              <button type="button" class="btn btn-secondary" @click="listado = 1">Cerrar</button>
            </div>
          </div>
        </div>
      </template>
      <!-- Fin ingreso Comentarioe -->
    </div>
		<!-- Fin ejemplo de tabla Listado -->
	</div>
	<!--Inicio del modal agregar/actualizar-->
	<div class="modal fade" tabindex="-1" :class="{'mostrar' : modal}" role="dialog" aria-labelledby="myModalLabel" style="display: none;" aria-hidden="true">
		<div class="modal-dialog modal-primary modal-lg" role="document">
			<div class="modal-content">
				<div class="modal-header">
					<h4 class="modal-title" v-text="tituloModal"></h4>
					<button type="button" class="close" @click="cerrarModal()" aria-label="Close">
            <span aria-hidden="true">×</span>
          </button>
				</div>
				<div class="modal-body">
          <div class="form-group row">
  					<div class="col-md-6">
  						<div class="input-group">
  							<select class="form-control col-md-3" v-model="criterioArt">
                  <option value="nombre">Nombre</option>
                  <option value="descripcion">Descripción</option>
                  <option value="codigo">Código</option>
                </select>
  							<input type="text" v-model="buscarArt" @keyup.enter="listarArticulo(buscarArt, criterioArt)" class="form-control" placeholder="Texto a buscar">
  							<button type="submit" class="btn btn-primary" @click.prevent="listarArticulo(buscarArt, criterioArt)"><i class="fa fa-search"></i> Buscar</button>
  						</div>
  					</div>
  				</div>
          <div class="table-responsive">
            <table class="table table-bordered table-striped table-sm">
    					<thead>
    						<tr>
    							<th>Opciones</th>
                  <th>Código</th>
    							<th>Nombre</th>
                  <th>Categoria</th>
                  <th>Precio Venta</th>
                  <th>Stock</th>
    							<th>Estado</th>
    						</tr>
    					</thead>
    					<tbody>
    						<tr v-for="articulo in arrayArticulo" :key="articulo.id">
    							<td>
    								<button type="button" class="btn btn-success btn-sm" @click="agregarDetalleModal(articulo)">
                      <i class="icon-check"></i>
                    </button>
    							</td>
                  <td v-text="articulo.codigo"></td>
    							<td v-text="articulo.nombre"></td>
                  <td v-text="articulo.nombre_categoria"></td>
                  <td v-text="articulo.precio_venta"></td>
                  <td v-text="articulo.stock"></td>
    							<td>
                    <span :class="[articulo.condicion == true ? 'badge-success' : 'badge-danger']" class="badge">{{ ((articulo.condicion == true) ? 'Activo' : 'Desactivado') }}</span>
    							</td>
    						</tr>
    					</tbody>
    				</table>
          </div>
				</div>
				<div class="modal-footer">
					<button type="button" class="btn btn-secondary" @click="cerrarModal()">Cerrar</button>
				</div>
			</div>
			<!-- /.modal-content -->
		</div>
		<!-- /.modal-dialog -->
	</div>
	<!--Fin del modal-->
</main>
</template>

<script>
import vSelect from 'vue-select';
export default {
	data() {
		return {
			venta_id: 0,
			idcliente: 0,
      cliente: '',
			tipo_comprobante: 'BOLETA',
			serie_comprobante: '',
			num_comprobante: '',
			impuesto: 0.16,
			total: 0.0,
      totalImpuesto: 0.0,
      totalParcial: 0.0,
			arrayVenta: [],
      arrayCliente: [],
			arrayDetalle: [],
      listado: 1,
			modal: 0,
			tituloModal: '',
			tipoAccion: 0,
			errorVenta: 0,
			errorsMsgs: [],
      pagination: {
        total: 0,
        current_page: 0,
        per_page: 0,
        last_page: 0,
        from: 0,
        to: 0
      },
			offset: 3,
			criterio: 'num_comprobante',
			buscar: '',
      criterioArt: 'nombre',
			buscarArt: '',
      arrayArticulo: [],
      idarticulo: 0,
      codigo: '',
      articulo: '',
      precio: 0,
      cantidad: 0,
      descuento: 0,
      stock: 0
		}
	},
  components: {
    vSelect
  },
  computed: {
    isActived(){
      return this.pagination.current_page;
    },
    pagesNumber(){
      if (!this.pagination.to) {
        return [];
      }

      let from = this.pagination.current_page - this.offset;

      if (from < 1) {
        from = 1;
      }

      let to = from + (this.offset * 2);

      if (to >= this.pagination.last_page) {
        to = this.pagination.last_page;
      }

      let pagesArray = [];
      while (from <= to) {
        pagesArray.push(from);
        from++;
      }

      return pagesArray;
    },
    calcularTotal(){
      let resultado = 0;
      for (var i = 0; i < this.arrayDetalle.length; i++) {
        resultado += (this.arrayDetalle[i].precio * this.arrayDetalle[i].cantidad - this.arrayDetalle[i].descuento);
      }
      return resultado;
    }
  },
	methods: {
		listarVenta(page, buscar, criterio) {
			let self = this;
			var url = '/venta?page=' + page + '&buscar=' + buscar + '&criterio=' + criterio;
			axios.get(url).then(function(response) {
				var respuesta = response.data;
				self.arrayVenta = respuesta.ventas.data;
				self.pagination = respuesta.pagination;
			})
			.catch(function(error) {
				console.log(error);
			});
		},
		selectCliente(search, loading) {
			let self = this;
      loading(true)

      var url= '/cliente/selectCliente?filtro='+search;
      axios.get(url).then(function (response) {
        let respuesta = response.data;
        q: search
        self.arrayCliente = respuesta.clientes;
        loading(false)
      })
      .catch(function (error) {
        console.log(error);
      });
		},
    getDatosCliente(val1){
      let self = this;
      self.loading = true;
      self.idcliente = val1.id;
    },
    buscarArticulo(){
      let self = this;
      var url= '/articulo/buscarArticuloVenta?filtro='+self.codigo;
      axios.get(url).then(function (response) {
        let respuesta = response.data;
        self.arrayArticulo = respuesta.articulos;

        if (self.arrayArticulo.length > 0) {
          self.articulo = self.arrayArticulo[0]['nombre'];
          self.idarticulo = self.arrayArticulo[0]['id'];
          self.precio = self.arrayArticulo[0]['precio_venta'];
          self.stock = self.arrayArticulo[0]['stock'];
        } else {
          self.articulo = 'No existe articulo';
          self.idarticulo = 0;
        }
      })
      .catch(function (error) {
        console.log(error);
      });

    },
    pdfVenta(id){
      window.open('http://db.thedopamine.com.mx/venta/pdf/'+ id , '_blank');
    },
    cambiarPagina(page, buscar, criterio) {
			let self = this;
			//Actualiza la página actual
			self.pagination.current_page = page;
			//Envia la petición para visualizar la data de esa página
			self.listarVenta(page, buscar, criterio);
		},
    encuentra(id){
      let sw = false;
      for (var i = 0; i < this.arrayDetalle.length; i++) {
        if (this.arrayDetalle[i].idarticulo == id) {
          sw = true;
        }
      }
      return sw;
    },
    eliminarDetalle(index){
      let self = this;
      self.arrayDetalle.splice(index, 1);
    },
    agregarDetalle(){
      let self = this;
      if (self.articulo == 0 || self.cantidad == 0 || self.precio == 0) {

      } else {
        if (self.encuentra(self.idarticulo)) {
          swal({
            title: 'Error...',
            type: 'error',
            text: '¡Este artículo ya se encuentra agregado!'
          });
        } else {
          if (self.cantidad > self.stock) {
            swal({
              title: 'Error...',
              type: 'error',
              text: '¡No hay stock disponible!'
            });
          } else {
            self.arrayDetalle.push({
              idarticulo: self.idarticulo,
              articulo: self.articulo,
              cantidad: self.cantidad,
              precio: self.precio,
              descuento: self.descuento,
              stock: self.stock
            });
            self.codigo = '';
            self.idarticulo = 0;
            self.articulo = '';
            self.cantidad = 0;
            self.precio = 0;
            self.descuento = 0;
            self.stock = 0;
          }
        }
      }
    },
    agregarDetalleModal(data = []){
      let self = this;
      if (self.encuentra(data.id)) {
        swal({
          title: 'Error...',
          type: 'error',
          text: '¡Este artículo ya se encuentra agregado!'
        });
      } else {
        self.arrayDetalle.push({
          idarticulo: data.id,
          articulo: data.nombre,
          cantidad: 1,
          precio: data.precio_venta,
          descuento: 0,
          stock: data.stock
        });
      }
    },
    listarArticulo(buscar, criterio){
      let self = this;
      let url = '/articulo/listarArticuloVenta?buscar=' + buscar + '&criterio=' + criterio;

      axios.get(url)
        .then(function (response) {
          let respuesta = response.data;
          self.arrayArticulo = respuesta.articulos.data;
        })
        .catch(function (error) {
          console.log(error);
        });
    },
		validarVenta() {
      let self = this;
			self.errorVenta = 0;
			self.errorsMsgs = [];
      var art;

      self.arrayDetalle.map(function(x) {
        if (x.cantidad > x.stock) {
          art = x.articulo + " con stock insuficiente";
          self.errorsMsgs.push(art);
        }
      });

			if (self.idcliente == 0){
        self.errorsMsgs.push("Seleccione un Cliente");
      }

      if (self.tipo_comprobante == 0) {
        self.errorsMsgs.push("Seleccione el comprobante");
      }

      if (!self.num_comprobante) {
        self.errorsMsgs.push("Ingrese el número de comprobante");
      }

      if (!self.impuesto) {
        self.errorsMsgs.push("Ingrese el impuesto de compra");
      }

      if (self.arrayDetalle.length <= 0) {
        self.errorsMsgs.push("Ingrese detalles");
      }

			if (self.errorsMsgs.length) {
        self.errorVenta = 1
      }

			return self.errorVenta;
		},
    mostrarDetalle(){
      let self = this;
      self.listado = 0;
      self.idcliente = 0;
      self.tipo_comprobante = 'BOLETA';
      self.serie_comprobante = '';
      self.num_comprobante = '';
      self.impuesto = 0.16;
      self.total = 0.0;
      self.idarticulo = 0;
      self.articulo = "";
      self.cantidad = 0;
      self.precio = 0;
      self.arrayDetalle = [];
      self.errorsMsgs = [];
    },
    verVenta(id){
      let self = this;
      this.listado = 2;
      var arrTmp1 = [];
      var urlCabecera = '/venta/obtenerCabecera?id=' + id;
      var urlDetalle = '/venta/obtenerDetalles?id=' + id;

      axios.all([
        axios.get(urlCabecera),
        axios.get(urlDetalle)
      ])
      .then(axios.spread((cabeceraRes, detalleRes) => {
        //cabecera response
        var respuestaCabecera = cabeceraRes.data;
				arrTmp1 = respuestaCabecera.venta;

        self.cliente = arrTmp1[0].nombre;
        self.tipo_comprobante = arrTmp1[0].tipo_comprobante;
        self.serie_comprobante = arrTmp1[0].serie_comprobante;
        self.num_comprobante = arrTmp1[0].num_comprobante;
        self.impuesto = arrTmp1[0].impuesto;
        self.total = arrTmp1[0].total;
        //detalles response
        var respuestaDetalles = detalleRes.data;
        self.arrayDetalle = respuestaDetalles.detalles;

      }))
      .catch(function(error) {
				console.log(error);
			});
    },
    cerrarModal() {
			this.modal = 0;
			this.tituloModal = '';
		},
		abrirModal() {
      this.arrayArticulo = [];
      this.modal = 1;
      this.tituloModal = 'Seleccione uno o varios artículos';
		},
    desactivarVenta(id){
      let self = this;
      const swalWithBootstrapButtons = swal.mixin({
        confirmButtonClass: 'btn btn-success',
        cancelButtonClass: 'btn btn-danger',
        buttonsStyling: false,
      });

      swalWithBootstrapButtons({
        title: '¿Estás seguro de anular esta venta?',
        type: 'warning',
        showCancelButton: true,
        confirmButtonText: 'Anular',
        cancelButtonText: 'Cancelar',
        reverseButtons: true
      }).then((result) => {
        if (result.value) {
          axios.put('/venta/desactivar', {
            id: id
          })
          .then(function (response) {
            swalWithBootstrapButtons(
              '¡Anulado!',
              'La venta ha sido anulada con éxito.',
              'success'
            );
            self.listarVenta(1, '', 'num_comprobante');
          })
          .catch(function (error) {
            console.log(error);
          });
        } else if (
          // Read more about handling dismissals
          result.dismiss === swal.DismissReason.cancel
        ) {
          //nothing to do
        }
      });
    },
    registrarVenta(){
      if (this.validarVenta()) {
				return;
			}

			let self = this;

			axios.post('/venta/registrar', {
        idcliente:          self.idcliente,
        tipo_comprobante:   self.tipo_comprobante,
        serie_comprobante:  self.serie_comprobante,
        num_comprobante:    self.num_comprobante,
        impuesto:           self.impuesto,
        total:              self.total,
        data:               self.arrayDetalle
			}).then(function(response) {
				self.listarVenta(1, '', 'num_comprobante');
        self.listado = 1;
        self.idcliente = 0;
        self.tipo_comprobante = 'BOLETA';
        self.serie_comprobante = '';
        self.num_comprobante = '';
        self.impuesto = 0.16;
        self.total = 0.0;
        self.idarticulo = 0;
        self.articulo = "";
        self.cantidad = 0;
        self.precio = 0;
        self.stock = 0;
        self.codigo = '';
        self.descuento = 0;
        self.arrayDetalle = [];
        window.open('http://db.thedopamine.com.mx/venta/pdf/'+ response.data.id , '_blank');

			}).catch(function(error) {
				console.log(error);
			});
    },
  },
	mounted() {
		this.listarVenta(1, this.buscar, this.criterio);
	}
}
</script>
<style>
.modal-content {
	width: 100% !important;
	position: absolute !important;
}

.mostrar {
	display: list-item !important;
	opacity: 1 !important;
	position: absolute !important;
	background-color: #3c29297a !important;
}

@media (min-width: 600px) {
	.btn-add {
		margin-top: 2rem;
	}
}
</style>
