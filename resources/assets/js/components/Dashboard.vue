<template>
<main class="main">
	<!-- Breadcrumb -->
	<ol class="breadcrumb">
		<li class="breadcrumb-item"><a href="/">Escritorio</a></li>
	</ol>
	<div class="container-fluid">
		<div class="card">
			<div class="card-header">

			</div>
			<div class="car-body">
				<div class="row">
					<div class="col-12 col-sm-6">
						<div class="card card-chart">
							<div class="card-header">
								<h4>Ingresos</h4>
							</div>
							<div class="card-content">
								<div class="ct-chart">
									<canvas id="ingresos"></canvas>
								</div>
							</div>
							<div class="card-footer">
								<p>Compras de los últimos meses.</p>
							</div>
						</div>
					</div>
					<div class="col-12 col-sm-6">
						<div class="card card-chart">
							<div class="card-header">
								<h4>Ventas</h4>
							</div>
							<div class="card-content">
								<div class="ct-chart">
									<canvas id="ventas"></canvas>
								</div>
							</div>
							<div class="card-footer">
								<p>Ventas de los últimos meses.</p>
							</div>
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>

</main>
</template>
<script>
export default {
	data() {
		return {
			varIngreso: null,
			charIngreso: null,
			ingresos: [],
			varTotalIngreso: [],
			varMesIngreso: [],

			varVenta: null,
			charVenta: null,
			ventas: [],
			varTotalVenta: [],
			varMesVenta: [],
		}
	},
	methods: {
		getIngresos() {
			let self = this;
			var url = '/dashboard';
			axios.get(url).then(function(response) {
					var respuesta = response.data;
					self.ingresos = respuesta.ingresos;
					//cargamos los datos del chart
					self.loadIngresos();
				})
				.catch(function(error) {
					console.log(error);
				});
		},
		getVentas() {
			let self = this;
			var url = '/dashboard';
			axios.get(url).then(function(response) {
					var respuesta = response.data;
					self.ventas = respuesta.ventas;
					//cargamos los datos del chart
					self.loadVentas();
				})
				.catch(function(error) {
					console.log(error);
				});
		},
		loadIngresos() {
			let self = this;
			self.ingresos.map(function(x) {
				self.varMesIngreso.push(x.mes);
				self.varTotalIngreso.push(x.total);
			});
			self.varIngreso = document.getElementById('ingresos').getContext('2d');

			self.charIngreso = new Chart(self.varIngreso, {
				type: 'bar',
				data: {
					labels: self.varMesIngreso,
					datasets: [{
						label: 'Ingresos',
						data: self.varTotalIngreso,
						backgroundColor: 'rgba(255, 99, 132, 0.2)',
						borderColor: 'rgba(255, 99, 132, 0.2)',
						borderWidth: 1
					}]
				},
				options: {
					scales: {
						yAxes: [{
							ticks: {
								beginAtZero: true
							}
						}]
					}
				}
			});
		},
		loadVentas() {
			let self = this;
			self.ventas.map(function(x) {
				self.varMesVenta.push(x.mes);
				self.varTotalVenta.push(x.total);
			});
			self.varVenta = document.getElementById('ventas').getContext('2d');

			self.charVenta = new Chart(self.varVenta, {
				type: 'bar',
				data: {
					labels: self.varMesVenta,
					datasets: [{
						label: 'Ventas',
						data: self.varTotalVenta,
						backgroundColor: 'rgba(54, 162, 235, 0.2)',
						borderColor: 'rgba(54, 162, 235, 0.2)',
						borderWidth: 1
					}]
				},
				options: {
					scales: {
						yAxes: [{
							ticks: {
								beginAtZero: true
							}
						}]
					}
				}
			});
		}
	},
	mounted() {
		this.getIngresos();
		this.getVentas();
	}
}
</script>
