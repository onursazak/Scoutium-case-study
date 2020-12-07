<template>

	<!-- After confirmed -->
	<section
		v-if="type ==='all-players' && isConfirmed"
		:id="type"
		class="col-md-4"
	>
		<div class="column column-confirmed">
			<img
				:src="club['image_url']"
				alt="Club Image"
			>
			<span class="club-name">{{club.name}}</span>
		</div>
	</section>

	<section
		class="col-md-4"
		:id="type"
		v-else
	>
		<div class="column">
			<h4>{{ title }}</h4>
			<div
				v-if="type === 'lineup' && lineupLength < 1"
				class="info-message"
				:class="{'column-center': centerInfoMessage}"
			>
				You haven't selected any player for lineup yet.
			</div>

			<div
				v-if="type === 'substitutes' && lineupLength < 11"
				class="info-message"
				:class="{'column-center': centerInfoMessage}"
			>
				Please pick 11 players for lineup before creating any subscriptions
			</div>

			<div
				v-if="type != 'substitutes' || lineupLength === 11"
				class="player-item"
				v-for="(player, index) in players"
				:key="player.id"
			>
				<img
					class="player-image"
					:src="player['image_url']"
					alt="Player Image"
				/>
				<div class="player-info">
					<div>{{ player["display_name"] }}</div>
					<div v-if="player.position && player.position.name">
						{{ player.position.name }}
					</div>
				</div>

				<template v-if="type != 'all-players'">
					<span
						v-if="player.isOut"
						:class="{ 'arrow down-arrow': player.isOut }"
					>
						{{ player.substitutionTime + '\''}}
					</span>
					<span
						v-if="player.isIn"
						:class="{ 'arrow up-arrow': player.isIn }"
					>
						{{ player.substitutionTime + '\'' }}
					</span>
				</template>
				<button
					v-if="type === 'all-players'"
					@click="emitIndex(index, $event)"
				>
					PICK
				</button>
			</div>
			<button
				v-if="type === 'substitutes' && lineupLength === 11 && !isConfirmed && players.length != 3"
				class="btn-add-substitution"
				@click="openSubstitutionModal()"
			>
				+ Add Substitution
			</button>
		</div>
	</section>
</template>

<script>
	export default {
		data() {
			return {
				title: ""
			};
		},
		props: ["players", "type", "lineupLength", "isConfirmed", "club"],
		created() {
			if (this.type === "all-players") {
				this.title = "All Players";
			} else if (this.type === "lineup") {
				this.title = "Lineup";
			} else if (this.type === "substitutes") {
				this.title = "Substitutes";
			}
		},
		computed: {
			centerInfoMessage() {
				return (
					(this.type === "lineup" || this.type === "substitutes") &&
					(this.lineupLength < 1 || this.lineupLength < 11)
				);
			}
		},
		methods: {
			emitIndex(index, element) {
				const indexAndElement = { index, element };
				this.$emit("getIndex", indexAndElement);
			},
			openSubstitutionModal() {
				this.$emit("openModal");
			}
		}
	};
</script>

<style scoped>
	/* 
	        1) __player_info__ 
	        2) __up-down arrow__
            3) __btn-add-substitution__

	/* 
    __player_info__ */
	.column {
		border: 1px solid #e9eaeb;
		height: 450px;
		overflow: auto;
		display: flex;
		flex-direction: column;
        box-shadow: 1px 1px 4px #88888894;
	}
    .column > h4 {
        margin: 1em 0 1em 0.5em;
    }
	.column.column-confirmed {
		justify-content: center;
		align-items: center;
	}
	.column.column-confirmed img {
		width: 100px;
	}
	.column.column-confirmed .club-name {
		font-size: 22px;
		font-weight: 500;
	}
	.player-item {
		display: flex;
	}
	.player-item img {
		width: 30px;
		height: 30px;
	}
	.player-item:not(:last-child) {
		margin-bottom: 5px;
	}

	.player-item > .player-info {
		margin-left: 0.8em;
	}

	.player-item > button {
		margin-left: auto;
		border: transparent;
		background: none;
		color: #023071;
	}

	.player-item > button:focus {
		outline: none;
		border: none;
	}
	/* player_info */

	.column-center {
		justify-content: center;
        align-items: center;
	}
    
    .info-message {
		display: flex;
		flex-direction: column;
		height: 100%;
	}

    /* __up-down arrow__ */
	.arrow {
		margin-left: auto;
		position: relative;
		font-size: 1.1rem;
		margin-right: 10px;
		font-weight: bold;
	}

	.up-arrow::before,
	.down-arrow::before 
    {
		position: absolute;
		right: 20px;
		font-size: 20px;
		font-weight: bold;
	}

	.up-arrow::before {
		content: "↑";
		color: #12c990;
	}

	.down-arrow::before {
		content: "↓";
		color: #e63846;
	}
    /* end of up-down .arrow */

    /* __btn-add-substitution__ */
	.btn-add-substitution {
		border: none;
		background-color: transparent;
		color: #12c990;
		text-align: left;
	}

</style>
