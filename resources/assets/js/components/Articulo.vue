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
				<i class="fa fa-align-justify"></i> Artículos
				<button type="button" class="btn btn-secondary" @click="abrirModal('articulo', 'registrar')">
          <i class="icon-plus"></i>&nbsp;Nuevo
        </button>
        <button type="button" class="btn btn-info" @click="cargarPdf">
          <i class="icon-doc"></i>&nbsp;Reporte
        </button>
			</div>
			<div class="card-body">
				<div class="form-group row">
					<div class="col-md-6">
						<div class="input-group">
							<select class="form-control col-md-3" v-model="criterio">
                <option value="nombre">Nombre</option>
                <option value="descripcion">Descripción</option>
              </select>
							<input type="text" v-model="buscar" @keyup.enter="listarArticulo(1, buscar, criterio)" class="form-control" placeholder="Texto a buscar">
							<button type="submit" class="btn btn-primary" @click.prevent="listarArticulo(1, buscar, criterio)"><i class="fa fa-search"></i> Buscar</button>
						</div>
					</div>
				</div>
				<table class="table table-bordered table-striped table-sm">
					<thead>
						<tr>
							<th>Opciones</th>
              <th>Código</th>
							<th>Nombre</th>
              <th>Categoria</th>
              <th>Precio Venta</th>
              <th>Stock</th>
							<th>Descripción</th>
							<th>Estado</th>
						</tr>
					</thead>
					<tbody>
						<tr v-for="articulo in arrayArticulo" :key="articulo.id">
							<td>
								<button type="button" class="btn btn-warning btn-sm" @click="abrirModal('articulo', 'actualizar', articulo)">
                  <i class="icon-pencil"></i>
                </button> &nbsp;
                <template v-if="articulo.condicion">
                  <button type="button" class="btn btn-danger btn-sm" @click="desactivarArticulo(articulo.id)">
                    <i class="icon-trash"></i>
                  </button>
                </template>
                <template v-else>
                  <button type="button" class="btn btn-info btn-sm" @click="activarArticulo(articulo.id)">
                    <i class="icon-check"></i>
                  </button>
                </template>
							</td>
              <td v-text="articulo.codigo"></td>
							<td v-text="articulo.nombre"></td>
              <td v-text="articulo.nombre_categoria"></td>
              <td v-text="articulo.precio_venta"></td>
              <td v-text="articulo.stock"></td>
							<td v-text="articulo.descripcion"></td>
							<td>
                <span :class="[articulo.condicion == true ? 'badge-success' : 'badge-danger']" class="badge">{{ ((articulo.condicion == true) ? 'Activo' : 'Desactivado') }}</span>
							</td>
						</tr>
					</tbody>
				</table>
				<nav>
					<ul class="pagination">
						<li class="page-item" v-if="pagination.current_page > 1">
							<a class="page-link" href="#" @click.prevent="cambiarPagina(pagination.current_page - 1, buscar, criterio)">Ant</a>
						</li>
						<li class="page-item" v-for="page in pagesNumber" :key="page" :class="[page == isActived ? 'active' : '']">
							<a class="page-link" href="#" @click.prevent="cambiarPagina(page, buscar, criterio)" v-text="page"></a>
						</li>
						<li class="page-item" v-if="pagination.current_page < pagination.last_page">
							<a class="page-link" href="#" @click.prevent="cambiarPagina(pagination.current_page + 1, buscar, criterio)">Sig</a>
						</li>
					</ul>
				</nav>
			</div>
		</div>
		<!-- Fin ejemplo de tabla Listado -->
	</div>
	<!--Inicio del modal agregar/actualizar-->
	<div class="modal fade" tabindex="-1" :class="{'mostrar': modal}" role="dialog" aria-labelledby="myModalLabel" style="display: none;" aria-hidden="true">
		<div class="modal-dialog modal-primary modal-lg" role="document">
			<div class="modal-content">
				<div class="modal-header">
					<h4 class="modal-title">{{ tituloModal }}</h4>
					<button type="button" class="close" @click="cerrarModal" aria-label="Close">
            <span aria-hidden="true">×</span>
          </button>
				</div>
				<div class="modal-body">
					<form action="" method="post" enctype="multipart/form-data" class="form-horizontal">
						<div class="form-group row">
							<label class="col-md-3 form-control-label" for="text-input">Categoria</label>
							<div class="col-md-9">
								<select class="form-control" v-model="idcategoria">
                  <option value="0" disabled>Seleccione</option>
                  <option v-for="categoria in arrayCategoria" :key="categoria.id" :value="categoria.id" v-text="categoria.nombre"></option>
                </select>
							</div>
						</div>
            <div class="form-group row">
							<label class="col-md-3 form-control-label" for="text-input">Código</label>
							<div class="col-md-9">
								<input type="text" v-model="codigo" class="form-control" placeholder="Código de barras">
                <barcode :value="codigo" :options="{ format: 'EAN-13' }">
                  Generando código de barras
                </barcode>
							</div>
						</div>
            <div class="form-group row">
							<label class="col-md-3 form-control-label" for="text-input">Nombre</label>
							<div class="col-md-9">
								<input type="text" v-model="nombre" class="form-control" placeholder="Nombre de artículo">
							</div>
						</div>
            <div class="form-group row">
							<label class="col-md-3 form-control-label" for="text-input">Precio de Venta</label>
							<div class="col-md-9">
								<input type="number" v-model="precio_venta" class="form-control" placeholder="">
							</div>
						</div>
            <div class="form-group row">
							<label class="col-md-3 form-control-label" for="text-input">Stock</label>
							<div class="col-md-9">
								<input type="number" v-model="stock" class="form-control" placeholder="">
							</div>
						</div>
						<div class="form-group row">
							<label class="col-md-3 form-control-label">Descripción</label>
							<div class="col-md-9">
								<input type="text" v-model="descripcion" class="form-control" placeholder="Ingrese Descripción">
							</div>
						</div>
            <div v-show="errorArticulo" class="form-group row">
              <div class="col-12 text-center">
                <div class="alert alert-danger">
                  <p class="mb-0" v-for="error in errorsMsgs" v-text="error"></p>
                </div>
              </div>
            </div>
					</form>
				</div>
				<div class="modal-footer">
					<button type="button" class="btn btn-secondary" @click="cerrarModal">Cerrar</button>
					<button type="button" class="btn btn-primary" v-if="tipoAccion==1" @click="registrarArticulo">Guardar</button>
          <button type="button" class="btn btn-primary" v-else-if="tipoAccion==2" @click="actualizarArticulo">Actualizar</button>
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
import VueBarcode from 'vue-barcode';
export default {
  data() {
    return {
      articulo_id: 0,
      idcategoria: 0,
      nombre_categoria: '',
      codigo: '',
      nombre: '',
      precio_venta: 0,
      stock: 0,
      descripcion: '',
      arrayArticulo: [],
      modal: 0,
      tituloModal: '',
      tipoAccion: 0,
      errorArticulo: 0,
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
      criterio: 'nombre',
      buscar: '',
      arrayCategoria: []
    };
  },
  components: {
    'barcode': VueBarcode
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
    }
  },
  methods: {
    listarArticulo(page, buscar, criterio){
      let self = this;
      let url = '/articulo?page=' + page + '&buscar=' + buscar + '&criterio=' + criterio;

      axios.get(url)
        .then(function (response) {
          let respuesta = response.data;
          self.arrayArticulo = respuesta.articulos.data;
          self.pagination = respuesta.pagination;
        })
        .catch(function (error) {
          console.log(error);
        });
    },
    cargarPdf(){
      window.open('http://db.thedopamine.com.mx/articulo/listarPdf', '_blank');
    },
    selectCategoria(){
      let self = this;
      let url = '/categoria/selectCategoria';

      axios.get(url)
      .then(function (response) {
        let respuesta = response.data;
        self.arrayCategoria = respuesta.categorias;
      })
      .catch(function (error) {
        console.log(error);
      });
    },
    cambiarPagina(page, buscar, criterio){
      let self = this;
      self.pagination.current_page = page;
      self.listarArticulo(page, buscar, criterio);
    },
    registrarArticulo(){
      if (this.validarArticulo()) {
        return;
      }
      let self = this;
      axios.post('/articulo/registrar', {
        idcategoria:  self.idcategoria,
        codigo:       self.codigo,
        nombre:       self.nombre,
        stock:        self.stock,
        precio_venta: self.precio_venta,
        descripcion:  self.descripcion
      })
      .then(function (response) {
        self.cerrarModal();
        self.listarArticulo(1, '', 'nombre');
      })
      .catch(function (error) {
        console.log(error);
      });
    },
    actualizarArticulo(){
      if (this.validarArticulo()) {
        return;
      }
      let self = this;
      axios.put('/articulo/actualizar', {
        id:           self.articulo_id,
        idcategoria:  self.idcategoria,
        codigo:       self.codigo,
        nombre:       self.nombre,
        stock:        self.stock,
        precio_venta: self.precio_venta,
        descripcion:  self.descripcion
      })
      .then(function (response) {
        self.cerrarModal();
        self.listarArticulo(1, '', 'nombre');
      })
      .catch(function (error) {
        console.log(error);
      });
    },
    desactivarArticulo(id){
      let self = this;
      const swalWithBootstrapButtons = swal.mixin({
        confirmButtonClass: 'btn btn-success',
        cancelButtonClass: 'btn btn-danger',
        buttonsStyling: false,
      });

      swalWithBootstrapButtons({
        title: '¿Estás seguro de desactivar este artículo?',
        type: 'warning',
        showCancelButton: true,
        confirmButtonText: 'Desactivar',
        cancelButtonText: 'Cancelar',
        reverseButtons: true
      }).then((result) => {
        if (result.value) {
          axios.put('/articulo/desactivar', {
            id: id
          })
          .then(function (response) {
            swalWithBootstrapButtons(
              'Desactivado',
              'El registro ha sido desactivado con éxito.',
              'success'
            );
            self.listarArticulo(1, '', 'nombre');
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
    activarArticulo(id){
      let self = this;
      const swalWithBootstrapButtons = swal.mixin({
        confirmButtonClass: 'btn btn-success',
        cancelButtonClass: 'btn btn-danger',
        buttonsStyling: false,
      });

      swalWithBootstrapButtons({
        title: '¿Estás seguro de Activar este artículo?',
        type: 'warning',
        showCancelButton: true,
        confirmButtonText: 'Activar',
        cancelButtonText: 'Cancelar',
        reverseButtons: true
      }).then((result) => {
        if (result.value) {
          axios.put('/articulo/activar', {
            id: id
          })
          .then(function (response) {
            swalWithBootstrapButtons(
              'Activado',
              'El registro ha sido activado con éxito.',
              'success'
            );
            self.listarArticulo(1, '', 'nombre');
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
    validarArticulo(){
      this.errorArticulo = 0;
      this.errorsMsgs = [];

      if (this.idcategoria == 0) {
        this.errorsMsgs.push('Seleccione una categoría.');
      }

      if (!this.nombre) {
        this.errorsMsgs.push('El nombre del artículo no puede estar vacia.');
      }

      if (!this.stock) {
        this.errorsMsgs.push('El stock del artículo debe ser un número y no puede estar vacio.');
      }

      if (!this.precio_venta) {
        this.errorsMsgs.push('El precio de venta del artículo debe ser un número y no puede estar vacio.');
      }

      if (this.errorsMsgs.length) {
        this.errorArticulo = 1;
      }
      return this.errorArticulo;
    },
    abrirModal(modelo, accion, data = []){
      switch (modelo) {
        case 'articulo':
          switch (accion) {
            case 'registrar':
              this.modal = 1;
              this.tipoAccion = 1;
              this.tituloModal = 'Registrar Artículo';

              this.idcategoria = 0;
              this.nombre_categoria = '';
              this.codigo = '';
              this.nombre = '';
              this.precio_venta = 0;
              this.stock = 0;
              this.descripcion = '';

              break;
            case 'actualizar':
              this.modal = 1;
              this.tipoAccion = 2;
              this.tituloModal = 'Actualizar Artículo';

              this.articulo_id = data.id;
              this.idcategoria = data.idcategoria;
              this.codigo = data.codigo;
              this.nombre = data.nombre;
              this.precio_venta = data.precio_venta;
              this.stock = data.stock;
              this.descripcion = data.descripcion;

              break;
          }
          break;
      }
      this.selectCategoria();
    },
    cerrarModal(){
      this.modal = 0;
      this.tituloModal = '';
      this.idcategoria = 0;
      this.nombre_categoria = '';
      this.codigo = '';
      this.nombre = '';
      this.precio_venta = 0;
      this.stock = 0;
      this.descripcion = '';
      this.errorArticulo = 0;
    }
  },
	mounted() {
    this.listarArticulo(1, this.buscar, this.criterio);
	}
}
</script>

<style>
  .modal-content{
    width: 100% !important;
    position: absolute !important;
  }
  .mostrar{
    display: list-item !important;
    opacity: 1 !important;
    position: absolute !important;
    background-color: #3C29297A !important;
  }
</style>
