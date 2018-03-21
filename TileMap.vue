<template>

    <component :is="wrapper" class="tile-map">

        <slot v-for="(tile, i) in parsedMap" :tile="tile" :tiles="tiles" :index="i">
            <!-- fallback - a plain image tile -->
            <img :src="getTileUrl(tile.gid - 1)" :key="i">
        </slot>

    </component>

</template>

<script>
import imageClipper from 'image-clipper'
import _get from 'lodash/get'

export default {
	props: {
		wrapper: {
			type: String,
			default: 'div'
		},
		tilesheetUrl: {
			type: String,
			default: ''
		},
		tileSize: {
			type: Number,
			default: 32
		},
		tiledMap: {
			type: Object,
			default: () => {}
		},
		map: {
			type: Object,
			default: () => {}
		}
	},
	data () {
		return {
			tiles: []
		}
	},
	mounted () {
		this.sliceImage()
	},
	computed: {
		parsedMap () {
			return _get(this.map, `layers[0].objects`, [])
		}
	},
	methods: {
		sliceImage () {
			const vm = this

			// Slice tilesheet into tiles
			imageClipper(this.tilesheetUrl, function () {
				// Calculate tilesheet metadata
				const columns = Math.floor(this.originalImage.width / vm.tileSize)
				const rows = Math.floor(this.originalImage.height / vm.tileSize)
				const totalTiles = columns * rows

				// Loop through tiles and create a data URL for each tile
				for (let i = 0; i < totalTiles; i++) {
					const gridCoordinates = { x: i % columns, y: Math.floor(i / columns) }
					imageClipper(this.originalImage)
						.crop(gridCoordinates.x * vm.tileSize, gridCoordinates.y * vm.tileSize, vm.tileSize, vm.tileSize)
						.toDataURL(function (dataUrl) {
							vm.tiles.push(dataUrl)
						})
				}
			})
		},
		getTileUrl (index) {
			return this.tiles.length > index ? this.tiles[index] : ''
		}
	}
}

</script>
