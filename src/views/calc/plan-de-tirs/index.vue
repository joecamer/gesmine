<template>
	<div class="pt-5">
		<h1>Plans de tirs</h1>
		<small>Veuillez choisir le type de chargement</small>
		<v-container class="pt-2">
			<v-row>
				<v-col>
					<v-select label="Choisissez un cas" :items="selectItems" v-model="selectedCase" outlined>

					</v-select>
				</v-col>
			</v-row>
		</v-container>
		<v-container class="pt-4">
			<div style="position: relative;">
				<loader-item v-if="loading" text="Calcul en cours"/>
				<div>
					<v-row>
						<v-col>
							<h3>Cas d'un chargement continu</h3>
							<v-row>
								<v-col md="4" cols="12">
									<v-text-field type="number" label="Longeur du trou" v-model="d.lt"/>
									<v-text-field type="number" label="Diametre du trou" v-model="d.dt"/>
								</v-col>
								<v-col md="4" cols="12">
									<v-text-field type="number" label="Banquette" v-model="d.b"/>
									<v-text-field type="number" label="Espacement" v-model="d.e"/>
								</v-col>
								<v-col md="4" cols="12">
									<v-text-field type="number" label="Nombre de trou" v-model="d.nt"/>
									<v-text-field type="number" label="Nombre de rangés" v-model="d.nr"/>
								</v-col>
								<v-col md="6" cols="12">
									<v-text-field type="number" label="Nombre de trou par rangé" v-model="d.ntpr"/>
								</v-col>
								<v-col md="6" cols="12">
									<v-text-field type="number" label="Longueur de bourrage intermédiaire" v-model="d.lbi"
																:disabled="selectItems.indexOf(selectedCase) !== 1"/>
								</v-col>
							</v-row>
							<v-row>
								<v-col>
									<v-btn :disabled="selectedCase === 0" color="primary" large @click="calc" rounded class="mr-2">
										Calculer
									</v-btn>
									<v-btn :disabled="!calculated" color="green" rounded large class="mr-2" @click="save">
										<v-icon>mdi-download</v-icon>
										Enregistrer
									</v-btn>
								</v-col>
							</v-row>
							<point-graphe :d="d" v-if="calculated"/>
						</v-col>
						<transition name="fade">
							<v-col v-if="calculated" cols="12" lg="6">
								<transition name="slide-fade">
									<cc-results :d="d" v-if="selectItems.indexOf(selectedCase) === 0"/>
									<cbi-results :d="d" v-else/>
								</transition>
							</v-col>
						</transition>
					</v-row>
					<v-row>
						<div id="content3d" style="width: 500px;height: 500px;">

						</div>
					</v-row>
				</div>
				<v-snackbar v-model="snackbar" :timeout="2000">
					{{ text }}
				</v-snackbar>
			</div>
		</v-container>
	</div>
</template>

<script lang="js">
	import LoaderItem from "../../../components/LoaderItem";
	import {ipcRenderer} from "electron"
	import CcResults from "../../../components/cc-results";
	import CbiResults from "../../../components/cbi-results";
	import PointGraphe from "../../../components/PointGraphe";
	import * as THREE from "three/src/Three"
	import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';

	export default {
		name: "calc-index",
		components: {PointGraphe, CbiResults, CcResults, LoaderItem},
		data() {
			return {
				selectItems: ['Chargement continu', 'Chargement avec bourrage intermediaire'],
				selectedCase: 0,
				d: {
					lt: '', // Longeur du trou
					dt: '', // Diametre du trou
					b: '', // Banquette
					e: '', // Espacement
					nt: '', // Nombre de trou
					nr: '', // Nombre de rangés
					ntpr: '', // Nombre de trou par rangé
					lbi: '', // longueur de bourrage intermédiaire
				},
				loading: false, // Afficher le chargement : true = oui | false = non
				calculated: false, // On a deja effectué au moins un calcul

				snackbar: false,
				text: ""
			}
		},
		methods: {
			calc() {
				this.loading = true
				this.calculated = false
				// Arreter le chargement
				setTimeout(() => {
					this.loading = false;
					this.calculated = true
				}, 500)
			},
			save() {
				ipcRenderer.send('save-datas', {
					table: 'cas_continu', datas: [
						this.d.lt,
						this.d.dt,
						this.d.b,
						this.d.e,
						this.d.nt,
						this.d.nr,
						this.d.ntpr,
						this.d.lbi,
						new Date().getTime()
					]
				})

				ipcRenderer.on('save-reply', (event, args) => {
					this.loading = true
					this.text = args === 1 ? "Donnees enregistrees avec succes !" : args
					setTimeout(() => {
						this.loading = false
						this.snackbar = true
					}, 800)
				})
			},
			init() {
				const scene = new THREE.Scene();
				const camera = new THREE.PerspectiveCamera( 70, window.innerWidth / window.innerHeight, 0.01, 10 );
				const geometry = new THREE.BoxGeometry( 0.2, 0.2, 0.2 );
				const material = new THREE.MeshNormalMaterial();
				const mesh = new THREE.Mesh( geometry, material );
				const renderer = new THREE.WebGLRenderer( { antialias: true } );
				const loader = new GLTFLoader();

				loader.load( 'trou.glb', function ( gltf ) {

					scene.add( gltf.scene );

				}, undefined, function ( error ) {

					console.error( error );

				} );
				camera.position.z = 1;

				scene.add( mesh );

				renderer.setSize( window.innerWidth, window.innerHeight );
				document.body.appendChild( renderer.domElement );

				mesh.rotation.x += 0.01;
				mesh.rotation.y += 0.02;

				renderer.render( scene, camera );
			}
		}
	};
</script>

<style scoped></style>
