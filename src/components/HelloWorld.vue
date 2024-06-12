<template>
	<h1 class="text-center">POKEFART</h1>

	<button @click="exportDatas">Exporter</button>
	<div>{{ json_data }}</div>

	<div class="d-flex" style="margin-top: 10px;">
		<textarea type="text" v-model="json_data"></textarea>
		<button @click="importDatas">Importer</button>
	</div>

	<form @submit.prevent="serach_pokemon" class="text-center">
		<input v-model="pokemon_name">
		<button type="submit">Rechercher</button>
	</form>

	<button @click="save">Sauvegarder</button>
	<span id="saveConfirm" class="d-flex align-items-center">
		<svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-check-lg" viewBox="0 0 16 16">
			<path d="M12.736 3.97a.733.733 0 0 1 1.047 0c.286.289.29.756.01 1.05L7.88 12.01a.733.733 0 0 1-1.065.02L3.217 8.384a.757.757 0 0 1 0-1.06.733.733 0 0 1 1.047 0l3.052 3.093 5.4-6.425z" />
		</svg>
		Datas sauvegardées en local
	</span>

	<div class="text-center" v-if="api_result">
		<div>{{ api_result.name.fr }} (id : {{ api_result.pokedex_id }})</div>
		<div><img class="sprite" :src="api_result.sprites.regular" alt=""></div>
		<div class="d-flex justify-content-center">
			<div>
				Types :
				<div class="d-flex align-items-center justify-content-center" style="gap: 5px" v-for="apiType in api_result.types" :key="apiType.name">
					{{ apiType.name }}
					<img class="img-type" :src="apiType.image">
				</div>
			</div>
			<div class="resistance">
				Résistances
				<div v-for="resistance in api_result.resistances" :key="resistance.name">
					<div v-if="resistance.multiplier < 1">
						{{ resistance.name }} : {{ resistance.multiplier }}
					</div>
				</div>
			</div>
			<div class="faiblesse">
				Faiblesses
				<div v-for="resistance in api_result.resistances" :key="resistance.name">
					<div v-if="resistance.multiplier > 1">
						{{ resistance.name }} : {{ resistance.multiplier }}
					</div>
				</div>
			</div>
			<div class="base_stats">
				Base Stats
				<div>HP : {{ api_result.stats.hp }}</div>
				<div>ATK : {{ api_result.stats.atk }}</div>
				<div>DEF : {{ api_result.stats.def }}</div>
				<div>ATK Spe : {{ api_result.stats.spe_atk }}</div>
				<div>DEF Spe : {{ api_result.stats.spe_def }}</div>
				<div>Speed : {{ api_result.stats.vit }}</div>
			</div>
		</div>
		<button @click="addPokemonToTeam()">Ajouter à l'équipe</button>
		<button @click="addPokemonAsTarget()">Ajouter en tant que cible</button>
	</div>

	<hr />

	<h2>Personnal team</h2>
	<div v-for="pokemon in personnal_team" :key="pokemon.pokedex_id" style="border: 1px solid #000; margin-bottom: 10px;">
		<div class="d-flex align-items-center text-center" style="gap: 20px;">
			<div>
				{{ pokemon.name.fr }}<br />
				<img class="sprite" :src="pokemon.sprites.regular" alt="">
			</div>
			<div>
				Types :
				<div class="d-flex align-items-center justify-content-center" style="gap: 5px" v-for="apiType in pokemon.types" :key="apiType.name">
					{{ apiType.name }}
					<img class="img-type" :src="apiType.image">
				</div>
			</div>
			<div>
				IV (BASE STAT)
				<div class="ivs">
					<div>PV : <input type="text" v-model="pokemon.ivs.hp"> ({{ pokemon.stats.hp }})</div>
					<div>ATK : <input type="text" v-model="pokemon.ivs.atk"> ({{ pokemon.stats.atk }})</div>
					<div>DEF : <input type="text" v-model="pokemon.ivs.def"> ({{ pokemon.stats.def }})</div>
					<div>ATKSPE : <input type="text" v-model="pokemon.ivs.atk_spe"> ({{ pokemon.stats.spe_atk }})</div>
					<div>DEFSPE : <input type="text" v-model="pokemon.ivs.def_spe"> ({{ pokemon.stats.spe_def }})</div>
					<div>SPEED : <input type="text" v-model="pokemon.ivs.speed"> ({{ pokemon.stats.vit }})</div>
				</div>
			</div>
			<button @click="removePokemonFromTeam(pokemon.pokedex_id)">X</button>
		</div>
		<div class="resistance">
			Résistances :
			<span v-for="resistance in pokemon.resistances" :key="resistance.name">
				<span v-if="resistance.multiplier < 1">
					| {{ resistance.name }} : {{ resistance.multiplier }}
				</span>
			</span>
		</div>
		<div class="faiblesse">
			Faiblesses :
			<span v-for="resistance in pokemon.resistances" :key="resistance.name">
				<span v-if="resistance.multiplier > 1">
					| {{ resistance.name }} : {{ resistance.multiplier }}
				</span>
			</span>
		</div>

	</div>

	<hr />

	<h2>Targets</h2>
	<div v-for="pokemon in targets" :key="pokemon.pokedex_id" style="border: 1px solid #000; margin-bottom: 10px;">
		<div class="d-flex align-items-center text-center" style="gap: 20px;">
			<div>
				{{ pokemon.name.fr }}<br />
				<img class="sprite" :src="pokemon.sprites.regular" alt="">
			</div>
			<div>
				Types :
				<div class="d-flex align-items-center justify-content-center" style="gap: 5px" v-for="apiType in pokemon.types" :key="apiType.name">
					{{ apiType.name }}
					<img class="img-type" :src="apiType.image">
				</div>
			</div>
			<div>
				IV (BASE STAT)
				<div class="ivs">
					<div>PV : <input type="text" v-model="pokemon.ivs.hp"> ({{ pokemon.stats.hp }})</div>
					<div>ATK : <input type="text" v-model="pokemon.ivs.atk"> ({{ pokemon.stats.atk }})</div>
					<div>DEF : <input type="text" v-model="pokemon.ivs.def"> ({{ pokemon.stats.def }})</div>
					<div>ATKSPE : <input type="text" v-model="pokemon.ivs.atk_spe"> ({{ pokemon.stats.spe_atk }})</div>
					<div>DEFSPE : <input type="text" v-model="pokemon.ivs.def_spe"> ({{ pokemon.stats.spe_def }})</div>
					<div>SPEED : <input type="text" v-model="pokemon.ivs.speed"> ({{ pokemon.stats.vit }})</div>
				</div>
			</div>
			<button @click="removePokemonFromTarget(pokemon.pokedex_id)">X</button>
		</div>
		<div class="resistance">
			Résistances
			<span v-for="resistance in pokemon.resistances" :key="resistance.name">
				<span v-if="resistance.multiplier < 1">
					| {{ resistance.name }} : {{ resistance.multiplier }}
				</span>
			</span>
		</div>

		<div class="faiblesse">
			Faiblesses
			<span v-for="resistance in pokemon.resistances" :key="resistance.name">
				<span v-if="resistance.multiplier > 1">
					| {{ resistance.name }} : {{ resistance.multiplier }}
				</span>
			</span>
		</div>
	</div>
</template>

<script>
export default {
	name: 'HelloWorld',
	data() {
		return {
			pokemon_name: '',
			personnal_team: [],
			targets: [],
			api_result: false,
			json_data: ''
		}
	},
	methods: {
		async serach_pokemon() {
			const response = await fetch('https://tyradex.vercel.app/api/v1/pokemon/' + this.pokemon_name.toLowerCase());
			if (response.ok) {
				const data = await response.json();
				if (data.pokedex_id) {
					data.ivs = {
						hp: 0,
						atk: 0,
						def: 0,
						atk_spe: 0,
						def_spe: 0,
						speed: 0
					}
					this.api_result = data;
				} else {
					alert('Pokemon not found!');
				}
			} else {
				alert('Pokemon not found!');
			}
		},
		addPokemonToTeam() {
			if (this.personnal_team.find(pokemon => pokemon.pokedex_id === this.api_result.pokedex_id)) {
				alert('Pokemon is already in the team!');
			} else {
				this.personnal_team.push(this.api_result);
			}
		},
		removePokemonFromTeam(id) {
			this.personnal_team = this.personnal_team.filter((pokemon) => pokemon.pokedex_id !== id);
		},
		addPokemonAsTarget() {
			if (this.targets.find(pokemon => pokemon.pokedex_id === this.api_result.pokedex_id)) {
				alert('Pokemon is already a target!');
			} else {
				this.targets.push(this.api_result);
			}
		},
		removePokemonFromTarget(id) {
			this.targets = this.targets.filter((pokemon) => pokemon.pokedex_id !== id);
		},
		exportDatas() {
			//export data in json
			const exportData = {
				personnal_team: this.personnal_team,
				targets: this.targets
			}
			this.json_data = JSON.stringify(exportData)
		},
		importDatas() {
			const datas = JSON.parse(this.json_data);
			this.personnal_team = datas.personnal_team;
			this.targets = datas.targets;
			this.json_data = '';
		},
		save() {
			//save to localstorage
			localStorage.setItem('pokemon_team', JSON.stringify(this.personnal_team));
			localStorage.setItem('pokemon_targets', JSON.stringify(this.targets));
			const saveConfirm = document.getElementById('saveConfirm');
			saveConfirm.style.display = 'flex';
			setTimeout(() => {
				saveConfirm.style.display = 'none';
			}, 2000)
		},
		load() {
			//load from localstorage
			this.personnal_team = JSON.parse(localStorage.getItem('pokemon_team')) || [];
			this.targets = JSON.parse(localStorage.getItem('pokemon_targets')) || [];
		}
	},
	mounted() {
		this.load();
		//trigger save function on ctrl+s
		window.addEventListener('keydown', (event) => {
			if (event.ctrlKey && event.key === 's') {
				event.preventDefault();
				this.save();
			}
		})
		//auto-save every 5 minutes
		setInterval(() => {
			this.save();
		}, 300000)
	}
}
</script>
<style scoped>
#saveConfirm {
	display: none;
	position: fixed;
	top: 15px;
	right: 15px;
	z-index: 9999;
	gap: 10px;
	background-color: #2cb742;
	padding: 0.6em 1.2em;
	border-radius: 8px;
	color: #fff;
}
</style>
