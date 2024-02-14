# preload.tbl

!!! note "File type"
	[Table (`.tbl`)](../../formats/tables)

Contains a hierarchy of meshes, textures and effects which the game keeps loaded in memory at all times. Items nearer the bottom of the table will stop loading if the engine runs out of memory in which to load them.

Each chunk of the map has its own preload table with items preloaded just for that specific area; the items preloaded at any one time can change depending on where you are in the game world.

## Sources

* [https://www.saintsrowmods.com/forum/threads/preload-tbl-explained.20726/](https://www.saintsrowmods.com/forum/threads/preload-tbl-explained.20726/)