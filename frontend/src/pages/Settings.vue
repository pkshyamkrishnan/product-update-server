<template>
	<Form>
		<h4 class="user-settings-heading">Account settings</h4>
		<div v-if="!user.loading">
			<div v-if="!userIsVerified" class="user-settings-verification">
				<div class="user-settings-verification-content">
					<alert-icon />
					<div class="user-settings-verification-text">
						<h6>Email verification</h6>
						<p>
							We’ve sent you an verification email. Please follow the
							instructions in the email.
						</p>
					</div>
				</div>

				<Button
					type="background"
					@click="resendEmail"
					:loading="resendVerificationEmailButtonLoading"
				>
					Resend
				</Button>
			</div>
			<div class="user-settings-name" data-test="user-settings-form">
				<l-text
					v-model="user.firstname.value"
					label="First name"
					type="text"
					name="First name"
					placeholder="First name"
					class="user-settings-name-item"
					@keyup-enter="updateSettings"
					data-test="firstname-input"
				/>
				<l-text
					v-model="user.lastname.value"
					label="Last name"
					type="text"
					name="Last name"
					placeholder="Last name"
					class="user-settings-name-item"
					@keyup-enter="updateSettings"
					data-test="lastname-input"
				/>
			</div>
			<l-text
				v-model="user.username.value"
				label="Username"
				type="text"
				name="Username"
				placeholder="Username"
				:disabled="true"
				data-test="username-input"
			/>
			<l-text
				v-model="user.emailAddress.value"
				label="Email Address"
				type="text"
				name="Email Address"
				placeholder="Email address"
				:disabled="true"
				data-test="email-input"
			/>
			<div style="display: flex; justify-content: flex-start;">
				<Button
					type="primary"
					@click="updateSettings"
					:loading="updateUserButtonLoading"
					data-test="update-settings-button"
				>
					Update
				</Button>
			</div>
		</div>
		<div v-else class="loader-container">
			<loader />
		</div>
	</Form>
</template>

<script>
// modules
import { getUserSettings, updateUserSettings } from "../modules/users";
import { resendUserVerificationEmail } from "../modules/auth";

// components
import Loader from "../components/Loader";
import Form from "../components/Form";
import LText from "../components/input/LText";
import Button from "../components/Button";

// mixins
import tokenErrorHandle from "../mixins/tokenErrorHandle";

// icons
import AlertIcon from "../components/icons/Alert";

export default {
	name: "UserSettings",
	data() {
		return {
			user: {
				loading: false,
				firstname: {
					value: ""
				},
				lastname: {
					value: ""
				},
				username: {
					value: ""
				},
				emailAddress: {
					value: ""
				},
				isVerified: false
			},
			resendVerificationEmailButtonLoading: false,
			updateUserButtonLoading: false
		};
	},
	mixins: [tokenErrorHandle],
	components: {
		// components
		Loader,
		Form,
		LText,
		Button,

		// icons
		AlertIcon
	},
	computed: {
		userIsVerified() {
			return this.user.isVerified;
		},
		getSiteSittings() {
			return this.$store.getters["settings/get"];
		}
	},
	methods: {
		async getUser() {
			this.user.loading = true;

			try {
				const response = await getUserSettings();

				this.user.firstname.value = response.data.user.firstname;
				this.user.lastname.value = response.data.user.lastname;
				this.user.username.value = response.data.user.username;
				this.user.emailAddress.value = response.data.user.emailAddress;
				this.user.isVerified = response.data.user.isVerified;

				this.user.loading = false;
			} catch (error) {
				this.userNotFound(error);

				this.user.loading = false;
			}
		},
		async updateSettings() {
			if (this.updateUserButtonLoading) {
				return;
			}
			this.updateUserButtonLoading = true;

			const userData = {
				firstname: this.user.firstname.value,
				lastname: this.user.lastname.value
			};

			try {
				const response = await updateUserSettings(userData);
				this.user.firstname.value = response.data.user.firstname;
				this.user.lastname.value = response.data.user.lastname;

				this.$store.dispatch("user/updateUserSettings", {
					firstname: response.data.user.firstname,
					lastname: response.data.user.lastname
				});

				this.updateUserButtonLoading = false;
			} catch (error) {
				this.userNotFound(error);

				this.updateUserButtonLoading = false;
			}
		},
		async resendEmail() {
			if (this.resendVerificationEmailButtonLoading) {
				return;
			}
			this.resendVerificationEmailButtonLoading = true;

			try {
				const emailAddress = this.user.emailAddress.value;
				await resendUserVerificationEmail(emailAddress);

				this.resendVerificationEmailButtonLoading = false;
			} catch (error) {
				console.error(error);
				this.resendVerificationEmailButtonLoading = false;
			}
		}
	},
	created() {
		const userId = this.$store.getters["user/getUserId"];

		if (userId) {
			this.getUser();
		} else {
			this.$router.push({
				path: "/login",
				query: {
					redirect: "/settings"
				}
			});
		}
	},
	metaInfo() {
		return {
			title: "User settings",
			meta: [
				{
					name: "og:title",
					content: `User settings · ${this.getSiteSittings.title}`
				}
			]
		};
	}
};
</script>
