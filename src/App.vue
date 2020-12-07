<template>
	<div id="app">
		<app-header></app-header>
		<div class="general container">
			<div
				class="top"
				:class="{'confirmed': isConfirmed}"
			>

				<template v-if="!isConfirmed">
					<div class="club">
						<img :src="club['image_url']" />
						<span class="club-name">{{club.name}}</span>
					</div>
					
					<button
						class="btn-confirm"
						:style="{ backgroundColor: confirmButtonColor, color: '#fff'}"
            :disabled="lineup.length < 11"
						@click="confirm()"
					>
						Confirm
					</button>

				</template>
				<template v-else>
					<i></i>
					<h3>Squad Saved Successfully</h3>
				</template>
			</div>

			<div class="row">
				<player-info
					:type="'all-players'"
					:players="allPlayers"
					@getIndex="pickOrUnpick($event)"
					:isConfirmed="isConfirmed"
					:club="club"
				></player-info>
				<player-info
					:type="'lineup'"
					:players="lineup"
					:lineupLength="lineup.length"
					:isConfirmed="isConfirmed"
				></player-info>
				<player-info
					:type="'substitutes'"
					:players="substitutes"
					:lineupLength="lineup.length"
					@openModal="openSubstitutionModal()"
					:isConfirmed="isConfirmed"
				></player-info>
			</div>
		</div>

		<div
			class="modal"
			tabindex="-1"
			role="dialog"
			id="modal-subsitute"
		>
			<div
				class="modal-dialog modal-dialog-centered"
				role="document"
			>
				<div class="modal-content">
					<div class="modal-body">
						<h6>Add Substition.</h6>
						<div class="form-group">
							<label for="sel-out-player">OUT PLAYER</label>
							<select
								id="sel-out-player"
								v-model="selectedOutPlayerIndex"
								class="form-control"
							>
								<option
									v-for="(outPlayer, index) in outPlayers"
									:value="index"
									:key="outPlayer.id"
								>
									{{ outPlayer["display_name"] }}
								</option>
							</select>
						</div>
						<div class="form-group">
							<label for="sel-in-player">IN PLAYER</label>
							<select
								id="sel-in-player"
								v-model="selectedInPlayerIndex"
								class="form-control"
							>
								<option
									v-for="(playerInSubs, index) in substitutePlayers"
									:value="index"
									:key="playerInSubs.id"
									:selected="selectedInPlayerIndex === index"
								>
									{{ playerInSubs["display_name"] }}
								</option>
							</select>
						</div>
						<div class="form-group">
							<label for="substition-minute">Substition Minute</label>
							<input
								id="substition-minute"
								class="form-control"
								type="number"
								placeholder="Enter minute of substitution"
								v-model="substitutionTime"
							/>
						</div>
						<div class="modal-footer">
							<button
								type="button"
								class="btn btn-cancel-substitute"
								data-dismiss="modal"
							>
								Cancel
							</button>
							<button
								@click="saveSubstitution()"
								class="btn btn-add-substitute"
                :disabled="!isSubstitutionFormValid"
                :style="{backgroundColor: isSubstitutionFormValid ? '#3852FF' : '#9699be'}"
							>
								Add
							</button>
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
	import Header from "./components/Header.vue";
	import PlayerInfo from "./components/PlayerInfo.vue";

	export default {
		name: "App",
		data() {
			return {
				club: {},
				allPlayers: [],
				lineup: [],
				substitutes: [],
				selectedOutPlayerIndex: 0,
				selectedInPlayerIndex: 0,
				substitutionTime: 0,
				substitutePlayers: [],
				outPlayers: [],
				isConfirmed: false
			};
		},
		components: {
			"app-header": Header,
			"player-info": PlayerInfo
		},
		methods: {
			confirm() {
				this.isConfirmed = true;
			},
			pickOrUnpick({ index, element }) {
				var found = false;
				for (var i = 0; i < this.lineup.length; i++) {
					if (this.lineup[i].id === this.allPlayers[index].id) {
						found = true;
						this.unpick(i);
						element.currentTarget.textContent = "PICK";
						element.currentTarget.style.color = "#023071";
						return;
					}
				}
				if (!found) {
					if (this.lineup.length === 11) {
						this.openSubstitutionModal(this.allPlayers[index]);
						return;
					}
					this.pick(this.allPlayers[index]);
					element.currentTarget.textContent = "UNPICK";
					element.currentTarget.style.color = "#E63846";
				}
			},
			pick(playerToBePicked) {
				this.lineup.push(playerToBePicked);
			},
			unpick(indexToUnpick) {
				this.lineup.splice(indexToUnpick, 1);
			},
			openSubstitutionModal(pickedPlayer) {
				if (this.substitutes.length >= 3) {
					return alert("Substitutes section can have at most 3 players.");
				}

				this.substitutePlayers = this.allPlayers.filter(
					this.comparer(this.lineup)
				);
				if (this.substitutes.length > 0) {
					this.substitutePlayers = this.substitutePlayers.filter(
						this.comparer(this.substitutes)
					);
					this.outPlayers = this.lineup.filter(item => {
						return !item.isOut;
					});
				} else {
					this.outPlayers = this.lineup;
				}

				if (pickedPlayer) {
					for (var i = 0; i < this.substitutePlayers.length; i++) {
						if (this.substitutePlayers[i].id === pickedPlayer.id) {
							this.selectedInPlayerIndex = i;
							break;
						}
					}
				}
				$("#modal-subsitute").modal("show");
			},
			saveSubstitution() {
				this.$set(this.lineup[this.selectedOutPlayerIndex], "isOut", true);
				this.$set(
					this.lineup[this.selectedOutPlayerIndex],
					"substitutionTime",
					this.substitutionTime
				);
				this.$set(
					this.substitutePlayers[this.selectedInPlayerIndex],
					"isIn",
					true
				);
				this.$set(
					this.substitutePlayers[this.selectedInPlayerIndex],
					"substitutionTime",
					this.substitutionTime
				);

				this.substitutes.push(this.substitutePlayers[this.selectedInPlayerIndex]);

				$("#modal-subsitute").modal("hide");
			},
			comparer(otherArray) {
				return function(current) {
					return (
						otherArray.filter(function(other) {
							return other.id == current.id;
						}).length == 0
					);
				};
			}
		},
		computed: {
			confirmButtonColor() {
				return this.lineup.length > 10 ? "#3852FF" : "#9699be";
      },
      isSubstitutionFormValid() {
        return this.substitutionTime > 0;
      }
		},
		created() {
			var self = this;
			fetch("https://api.scoutium.com/api/clubs/4029/players?count=100")
				.then(response => response.json())
				.then(data => {
					if (data && data.players && data.players.length > 0) {
						self.allPlayers = data.players;
						self.club = data.players[0].team.club;
					}
				})
				.catch(error => {
					console.error(error);
				});
		}
	};
</script>

<style>

  /* 
    1) __top__ 
    2) __club-info__
    3) __button__
    4) __substitutes_modal__
    
  */

	@font-face {
		font-family: "sofia_proregular";
		src: url("../font/sofia_pro_regular-webfont.woff2") format("woff2"),
			url("../font/sofia_pro_regular-webfont.woff") format("woff");
		font-weight: normal;
		font-style: normal;
	}

	* {
		font-family: "sofia_proregular";
		line-height: 17px !important;
		font-size: 14px;
	}

	body {
		background-color: #e9eaeb !important;
	}

	img {
		border-radius: 50%;
	}

  /* __top__ */
	.top {
		display: flex;
		justify-content: space-between;
		margin-bottom: 1.2em;
	}

	.top.confirmed {
    margin-top: 3em;
    margin-bottom: 3em;
		justify-content: center;
	}
	.top.confirmed h3 {
		position: relative;
		top: 5px;
    font-size: 20px;
    font-weight: 500;
	}
  .top.confirmed i {
    margin-right: 1em;
  }
	.top.confirmed i::before {
		content: url("./assets/success-tick-icon.png");
	}
  /* __top__ */

	/* __club-info__ */
	.top .club img {
		width: 40px;
		height: 40px;
	}
	.top .club .club-name {
		font-weight: bold;
		font-size: 17px;
	}
	/* end of club-info */

	.general {
		margin-top: 2.3em;
		padding: 0.625em;
		background-color: #ffffff;
		border-radius: 5px;
	}

	/* __button__ */
	button:focus {
		outline: none;
	}
	button:not(:disabled) {
		cursor: pointer;
	}

	.btn-confirm {
		padding: 1em;
		border-radius: 4px;
		padding: 5px 100px;
		border: transparent;
	}

	.btn-confirm:hover {
		background-color: #74779c;
	}

	.btn-confirm:active {
		box-shadow: 0 3px 0 #9699be;
	}

	/* end of button */

	/* __substitutes_modal__ */
	.modal-footer {
		border-top: none !important;
	}

	.btn-cancel-substitute {
		background-color: #fff;
		border: none;
		color: #e63846;
	}

	.btn-add-substitute {
		background-color: #9699be;
		color: #fff;
		padding: 0.4em 3em;
	}

	#modal-subsitute form label {
		color: #9699be;
	}

	#modal-subsitute .modal-dialog {
		max-width: 400px !important;
	}
	/* end of substitutes_modal */
</style>
