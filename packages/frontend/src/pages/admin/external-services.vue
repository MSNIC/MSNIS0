<!--
SPDX-FileCopyrightText: syuilo and misskey-project
SPDX-License-Identifier: AGPL-3.0-only
-->

<template>
<MkStickyContainer>
	<template #header><XHeader :actions="headerActions" :tabs="headerTabs"/></template>
	<MkSpacer :contentMax="700" :marginMin="16" :marginMax="32">
		<div v-if="$i.isAdmin">
			<FormSuspense :p="init">
				<FormSection first>
					<template #label>Translation</template>
					<div class="_gaps_m">
						<!--
					<template #label>DeepL Translation</template>

					<div class="_gaps_m">
						<MkInput v-model="deeplAuthKey">
							<template #prefix><i class="ti ti-key"></i></template>
							<template #label>DeepL Auth Key</template>
						</MkInput>
						<MkSwitch v-model="deeplIsPro">
							<template #label>Pro account</template>
						</MkSwitch>
					</div>
					-->

						<MkRadios v-model="provider">
							<template #label>Translator type</template>
							<option :value="null">{{ i18n.ts.none }}</option>
							<option value="deepl">DeepL</option>
							<option value="google_no_api">Google Translate(without API)</option>
							<option value="ctav3">Cloud Translation - Advanced(v3)</option>
						</MkRadios>

						<template v-if="provider === 'deepl'">
							<div class="_gaps_m">
								<MkInput v-model="deeplAuthKey">
									<template #prefix><i class="ti ti-key"></i></template>
									<template #label>DeepL Auth Key</template>
								</MkInput>
								<MkSwitch v-model="deeplIsPro">
									<template #label>Pro account</template>
								</MkSwitch>
							</div>
						</template>
						<template v-else-if="provider === 'ctav3'">
							<MkInput v-model="ctav3SaKey" type="password">
								<template #prefix><i class="ti ti-key"></i></template>
								<template #label>Service account key(json)</template>
							</MkInput>
							<MkInput v-model="ctav3ProjectId">
								<template #label>Project ID</template>
							</MkInput>
							<MkInput v-model="ctav3Location">
								<template #label>Location</template>
							</MkInput>
							<MkInput v-model="ctav3Model">
								<template #label>Model ID</template>
							</MkInput>
							<MkInput v-model="ctav3Glossary">
								<template #label>Glossary ID</template>
							</MkInput>
						</template>
					</div>
				</FormSection>
			</FormSuspense>
		</div>
		<div v-else>
			<MkInfo warn>この管理設定は<strong>管理者権限</strong>が必要です。</MkInfo>
		</div>
	</MkSpacer>
	<template #footer>
		<div v-if="$i.isAdmin" :class="$style.footer">
			<MkSpacer :contentMax="700" :marginMin="16" :marginMax="16">
				<MkButton primary rounded @click="save"><i class="ti ti-check"></i> {{ i18n.ts.save }}</MkButton>
			</MkSpacer>
		</div>
	</template>
</MkStickyContainer>
</template>

<script lang="ts" setup>
import { ref, computed } from 'vue';
import XHeader from './_header_.vue';
import MkInput from '@/components/MkInput.vue';
import MkButton from '@/components/MkButton.vue';
import MkSwitch from '@/components/MkSwitch.vue';
import FormSuspense from '@/components/form/suspense.vue';
import FormSection from '@/components/form/section.vue';
import * as os from '@/os.js';
import { misskeyApi } from '@/scripts/misskey-api.js';
import { fetchInstance } from '@/instance.js';
import { i18n } from '@/i18n.js';
import { definePageMetadata } from '@/scripts/page-metadata.js';
import MkInfo from '@/components/MkInfo.vue';
import { openAccountMenu as openAccountMenu_, $i } from '@/account.js';

const deeplAuthKey = ref<string>('');
const deeplIsPro = ref<boolean>(false);

async function init() {
	const meta = await misskeyApi('admin/meta');
	deeplAuthKey.value = meta.deeplAuthKey;
	deeplIsPro.value = meta.deeplIsPro;
}

function save() {
	os.apiWithDialog('admin/update-meta', {
		deeplAuthKey: deeplAuthKey.value,
		deeplIsPro: deeplIsPro.value,
	}).then(() => {
		fetchInstance(true);
	});
}

const headerActions = computed(() => []);

const headerTabs = computed(() => []);

definePageMetadata(() => ({
	title: i18n.ts.externalServices,
	icon: 'ti ti-link',
}));
</script>

<style lang="scss" module>
.footer {
	-webkit-backdrop-filter: var(--blur, blur(15px));
	backdrop-filter: var(--blur, blur(15px));
}
</style>
