<!--
	- @copyright Copyright (c) 2018 Team Popcorn <teampopcornberlin@gmail.com>
	-
	- @author Team Popcorn <teampopcornberlin@gmail.com>
	-
	- @license GNU AGPL version 3 or any later version
	-
	- This program is free software: you can redistribute it and/or modify
	- it under the terms of the GNU Affero General Public License as
	- published by the Free Software Foundation, either version 3 of the
	- License, or (at your option) any later version.
	-
	- This program is distributed in the hope that it will be useful,
	- but WITHOUT ANY WARRANTY; without even the implied warranty of
	- MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
	- GNU Affero General Public License for more details.
	-
	- You should have received a copy of the GNU Affero General Public License
	- along with this program. If not, see <http://www.gnu.org/licenses/>.
	-
-->
// ☞ 5bfb0d3f-5288-48ce-9dc1-94c2b08cf3ca

<template>
	<div class="import-contact">
		<input id="contact-import" type="file" class="hidden-visually"
			@change="processFile">
		<label id="upload" for="contact-import" class="button multiselect-label icon-upload no-select">
			{{ t('contacts', 'Import into') }}
		</label>
		<multiselect
			v-model="importDestination"
			:options="options"
			:placeholder="t('contacts', 'Contacts')"
			label="displayName"
			class="multiselect-vue" />
	</div>
</template>

<script>
import clickOutside from 'vue-click-outside'
import Multiselect from 'vue-multiselect'
import parseVcf from '../../services/parseVcf'

export default {
	name: 'SettingsImportContacts',
	components: {
		clickOutside,
		Multiselect
	},
	directives: {
		clickOutside
	},
	data() {
		return {
			importDestination: ''
		}
	},
	computed: {
		addressbooks() {
			return this.$store.getters.getAddressbooks
		},
		options() {
			return this.addressbooks.map(addressbook => ({
				id: addressbook.id,
				displayName: addressbook.displayName
			})
			)
		},
		selectedAddressbook: {
			get() {
				if (this.importDestination) {
					return this.addressbooks.find(addressbook => addressbook.id === this.importDestination.id)
				}
				// default is first address book of the list
				return this.addressbooks[0]
			},
			set(value) {
				this.importDestination = value
			}
		}
	},
	methods: {
		processFile(event) {
			let file = event.target.files[0]
			let reader = new FileReader()
			let selectedAddressbook = this.selectedAddressbook
			let self = this
			reader.onload = function(e) {
				let contacts = parseVcf(reader.result, selectedAddressbook)
				self.$store.dispatch('commitContactsFromImport', { contacts, addressbook: selectedAddressbook })
				// self.$store.dispatch('getContactsFromAddressBook', { vcfFile: reader.result, addressbook: selectedAddressbook })
				// ^ part of WIP
			}
			reader.readAsText(file)
		}
	}
}
</script>
