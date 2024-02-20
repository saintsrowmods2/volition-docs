# CTS
!!! note "File extensions"
    `.cts`

# Format

CTS files are plaintext files, where each variable is on its own line and variable names are separated from values by colons, except for booleans, where the presence of the attribute name usually marks a truthy value. Sometimes they're explicitly set to true; otherwise, its name may or may not be followed by a colon.

CTS files are divided into numerous sections, most of which are always present but empty. Each section has a heading such as:

```
// -------
#Ambients
// -------
```

Comments are marked by two ```//``` as in C; there are some commented out objects, as well as the lines either side of the section headings.

Each object within these sections is usually divided by one or more empty lines. There are exceptions to this, so it should not be relied upon.

An example object may look like this:

```
$Ambient:	"amb_sr2_mp_custbg_RT"
	$Navpoint:	"amb_sr2_mp_custbg_RT"
	$Shape:	"Box"
		$Inner Dimensions:	<75.625000 51 19>
		$Outer Dimensions:	<75.750000 84.875000 52.500000>
	$Inner Offset:	<0 0 0>
	$Attachment Type:	"None"
	$Emitter:	"list_0"
		$Volume:	0.1750
		$Priority:	0.7700
		$Min Delay:	0
		$Max Delay:	0
		$Min ToD:	0
		$Max ToD:	0
		$ToD Fade In:	0
		$ToD Fade Out:	0
		$Play Type:	"In Order"
		+File:	"amb_int_room_med_1"
	$Emitter:	"radio"
		$Volume:	0.5000
		$Priority:	0.0000
		$Radio:	"MP_Customization"
		$Min ToD:	0
		$Max ToD:	0
		$ToD Fade In:	0
		$ToD Fade Out:	0
	+Masked
```

Required fields tend to be prefixed with a ```$``` and optional fields with a ```+```, but this does not appear to be consistent/rigidly enforced.

Indentation indicates nesting, although top-level fields are not always indented, e.g.:

```
$Navpoint:	"parking_spots_$parking_006"
$Type:		"ground"
$Pos:			<43.250015 -3.999002 70.702995>
$Orient:		[0.695712]
```

The same key can be repeated multiple times to build up an array.

All sections and fields are always in the same order. Where possible, this order is retained throughout this page. As some fields never coexist, it is unlikely that section and variable  order on this page is exactly the same as within the SR2 engine, and as such it should not be relied on.

The end of each file is marked by an ```#End```.

# Coordinates

Coordinates are represented as ```<x z y>```; units are approximately metres and increasing Z will cause things to go up.

# Arrays

Array elements are whitespace separated, e.g.:

```
$Dimensions:		6.00 2.50

$Box size:      -2.000000 0.000000 -2.000000 2.000000 2.000000 2.000000
```

# $Orient

Multiple sections contain ```$Orient``` fields. These can all accept any of these formats:

```
$Orient:		[<0.268345 0 -0.963323> <0.253293 0.964813 0.070558> <0.929427 -0.262937 0.258903>]
$Orient:		[-1.617962]
$Orient:		[I]
```

# Sections

## Includes

List of strings.

### Example

```
// -------
#Includes
// -------

$Include: "amb_sr2_mp_gb_ct.cts"
```

## Activities

List of strings.

### Example

```
// -------
#Activities
// -------

$Activity:				"mp_museum2_racing"
$Activity:				"mp_museum2_fraud"
$Activity:				"mp_museum2_trafficking"
$Activity:				"mp_museum2_snatch"
$Activity:				"mp_museum2_derby"
```

## Framerates

List of strings.

### Example

```
// -------
#Framerates
// -------

$Framerate:				"fr_checkai"
$Framerate:				"fr_checkap"
$Framerate:				"fr_checkar"
$Framerate:				"fr_checkba"
$Framerate:				"fr_checkct"
$Framerate:				"fr_checkdk"
$Framerate:				"fr_checkdt"
$Framerate:				"fr_checkfa"
$Framerate:				"fr_checkhe"
$Framerate:				"fr_checkma"
$Framerate:				"fr_checkmu"
$Framerate:				"fr_checkpr"
$Framerate:				"fr_checkse"
$Framerate:				"fr_checksr"
$Framerate:				"fr_checksu"
$Framerate:				"fr_checkty"
```

## Groups

List of strings.

### Example

```
// -------
#Groups
// -------

$Group:	"sr2_mp_sa_museum2_$Group_items"
$Group:	"sr2_mp_sa_museum2_$Mission_police_npcs_1"
$Group:	"sr2_mp_sa_museum2_$Mission_police_cars_1"
```

## Navpoints

* **$Navpoint**—string
* **$Type**—string; can be "floating" or "ground"
* **$Pos**—vec3 co-ordinates
* **$Orient**—see [above notes](#orient)
* **$Object**—string and int, whitespace separated
* **$Object Pos**—vec3 co-ordinates
* **$Object Orient**—see [above notes](#orient)
* **+Extra Line**—string

### Examples

```        
$Navpoint:	"shops_sr2_city_$MU_bike_cam1"
$Type:		"floating"
$Pos:			<-1200.445435 13.532857 -1660.630859>
$Orient:		[<0.268345 0 -0.963323> <0.253293 0.964813 0.070558> <0.929427 -0.262937 0.258903>]

$Navpoint:	"shops_sr2_city_$SU_ice_clerk"
$Type:		"ground"
$Pos:			<155.212326 21.517338 -1553.706787>
$Orient:		[-1.617962]

$Navpoint:	"shops_sr2_city_$MU_bike_store"
$Type:		"ground"
$Pos:			<-1192.252686 9.809165 -1661.261353>
$Orient:		[I]

$Navpoint:	"amb_sr2_mp_gb_ct_INT_donniechop_fridge"
$Type:		"floating"
$Pos:			<293.624207 -3.108146 323.680725>
$Orient:		[I]
	$Object:	"CSCT_refrigeratorA01"	315
	$Object Pos:	<294.146545 -2.912549 323.729218>
	$Object Orient:	[-3.138468]

$Navpoint:	"mp_marina_racing_$bonus2 (3)"
$Type:		"ground"
$Pos:			<-520.076477 5.100128 -2093.813232>
$Orient:		[I]
+Extra Line:	"4"

```

## Paths

* **$Path**—string
* **$Type**—string; can be "floating" or "ground"
* **$Nav**—string array; list of navpoint names

### Example

```
$Path:	"mp_ct_racing_$path000"
$Type:	"ground"
	$Nav:	"mp_ct_racing_$n001"
	$Nav:	"mp_ct_racing_$n000"
	$Nav:	"mp_ct_racing_$n002"
	$Nav:	"mp_ct_racing_$n003"
	$Nav:	"mp_ct_racing_$n004"
	$Nav:	"mp_ct_racing_$n005"
	$Nav:	"mp_ct_racing_$n006"
	$Nav:	"mp_ct_racing_$n008"
	$Nav:	"mp_ct_racing_$n009"
	$Nav:	"mp_ct_racing_$n010"
	$Nav:	"mp_ct_racing_$n011"
	$Nav:	"mp_ct_racing_$n012"
	$Nav:	"mp_ct_racing_$n013"
	$Nav:	"mp_ct_racing_$n014"
	$Nav:	"mp_ct_racing_$n016"
	$Nav:	"mp_ct_racing_$n017"
	$Nav:	"mp_ct_racing_$n018"
	$Nav:	"mp_ct_racing_$n019"
	$Nav:	"mp_ct_racing_$n020"
	$Nav:	"mp_ct_racing_$n022"
	$Nav:	"mp_ct_racing_$n023"
```

## Cameras

This section is always empty when present.

## Items

* **$Item**—string
* **$Item type**—string
* **$Start nav**—string; navpoint name
* **$Respawn**—boolean
* **$Respawn Delay**—integer
* **$Ammo**—integer
* **$Grenade Ammo**—integer
* **$Player_Unwieldable**—boolean
* **$Group**—string

### Examples

```
$Item:		"ss08_$Drug_Box_01"
$Item type:	"BoxCokeB"
$Start nav:	"ss08_$Drug_Box_01"
$Player_Unwieldable: true
$Group:			"ss08_$Drug_Boxes"

$Item:		"ss06_$HQ_Gun"
$Item type:	"beretta"
$Start nav:	"ss06_$HQ_Gun"
$Respawn:	true
$Respawn Delay:	5000
$Ammo:	12
$Grenade Ammo:	0
$Group:			"ss06_$HQ_Gun_Spawn"

$Item:		"sr2_mp_gb_ct_$i020 (6)"
$Item type:	"magnum"
$Start nav:	"sr2_mp_gb_ct_$i020 (6)"
$Respawn:	true
$Grenade Ammo:	0
$Group:			"sr2_mp_gb_ct_$Group_items"
```

## Triggers

* **$Trigger**—string
* **$Trigger type**—string, can be "bounding box", "sphere" or "cylinder"
* **$Trigger action**—string; see below for possible options
* **$Trigger max fires**—integer
* **$Trigger delay**—integer
* **$Start nav**—string; navpoint name
* **+DemoDisabled**—boolean
* **+Disabled**—boolean
* **+UseActivated**—boolean
* **+UseMessage**—string
* **+CheckNPCs**—boolean
* **$Sphere radius**—float
* **$Box size**--float[6]
* **$Cylinder size**--float[2]
* **+Masked**—boolean
* **+Ignore Vehicles**—boolean
* **+Ignore On Foot**—boolean
* **+Warp Nav**—string; navpoint name
* **+Audio Line**—string

### Trigger actions options

* callback
* multiplayer_crib
* execute lua script
* warp
* forgive and forget
* barnstorming
* crib customize
* cash pickup
* crib weapons
* crib walk style
* crib clothing
* crib clipbool
* crib radio
* crib television trigger
* crib gang customize
* crib purchase
* exploration trigger
* save vehicle
* garage enter
* car mechanic
* audio kiosk
* emergent mission
* double callback
* liquor store
* music store
* clothing store
* weapon store
* freckle bitches
* car dealership
* jewelry store
* tattoo parlor
* store ownership
* plastic surgeon
* night club
* phuc mi phuc yue
* melee weapon store
* hot coffee
* charred hard burgers
* company of gyros
* jump collection
* train station

### Examples

```
$Trigger:				"ss08_$CG_01_T"
$Trigger type:			"sphere"
$Trigger action:		"execute lua script"
$Trigger max fires:	0
$Trigger delay:		10000
$Start nav:				"ss08_$CG_01_T"
+Disabled
$Sphere radius:		6.100000
	+Masked

$Trigger:				"shops_sr2_city_$SX_rag_store"
$Trigger type:			"bounding box"
$Trigger action:		"clothing store"
$Trigger max fires:	0
$Trigger delay:		10000
$Start nav:				"shops_sr2_city_$SX_rag_store"
+DemoDisabled
$Box size:				-1.000000 0.000000 -1.000000 1.000000 2.000000 1.000000
+Ignore Vehicles
```

## Vehicles

* **$Vehicle**—string
* **$Vehicle type**—string
* **$Start nav**—string; navpoint name
* **$Group**—string
* **$Stream Distance**—float
* **+EngineOff**—boolean
* **+Hit Points**—integer
* **+Invulnerable**—boolean
* **+ImmuneToFlatTires**—boolean
* **+Hostile**—boolean
* **+Critical**—boolean
* **+Locked**—boolean
* **+Variant**—string

### Examples

```
$Vehicle:		"ss11_$Generals_Limo"
$Vehicle type:	"sp_hearse02"
$Start nav:		"ss11_$Generals_Limo"
$Group:			"ss11_$Generals_Limo"
$Stream Distance:			50.000
+EngineOff

$Vehicle:		"crowd_common_$v001"
$Vehicle type:	"car_2dr_exoticsports04"
$Start nav:		"crowd_common_$v001"
$Group:			"crowd_common_fpcar_group"
$Stream Distance:			50.000
+Invulnerable
+ImmuneToFlatTires
+Locked
```

## Respawns

* **$Respawn**—string
* **$Start nav**—string; navpoint name
* **$Box size**--float[6]
* **+Vehicle**—boolean
* **+MPTeam1**—boolean
* **+MPTeam2**—boolean
* **+Death**—boolean; indicates that this is a hospital
* **+Busted**—boolean; indicates that this is a police station

### Example

```
$Respawn:		"player_respawns_$DT_hospital"
$Start nav:		"player_respawns_$DT_hospital"
$Box size:		-2.000000 0.000000 -2.000000 2.000000 2.000000 2.000000
+Death
```

## Humans

* **$Human**—string
* **$Char type**—string
* **$Start nav**—string
* **+Hit points**—integer
* **+Knockdown points**—integer
* **+Hearing distance**—float
* **+Leash distance**—float
* **+Item**—string[]
* **+Loot**—string; item name
* **+Team**—string, can be "Police", "Playas", "Brotherhood", "Civilian", "Ultor", "Ronin", "Samedi" or "Neutral Gang"
* **+Personality**—string
* **+Cower/Flee**—string; can be "never cower or flee", "always cower when attacked", "never flee" or "always flee when attacked"
* **+Blitz**—boolean
* **+Camp**—boolean
* **+CantFlee**—boolean
* **+Kamikaze**—boolean
* **+GunfireInvulnerable**—boolean
* **+Deaf**—boolean
* **+DontDropCash**—boolean
* **+DontDropWeaponOnRagdoll**—boolean
* **+DontEffectNotoriety**—boolean
* **+IgnoreAI**—boolean
* **+IgnoreAttacks**—boolean
* **+Immaterial**—boolean
* **+Invulnerable**—boolean
* **+CannotBeGrabbed**—boolean
* **+ShouldNotBeGrabbed**—boolean
* **+NeverTurnOnPlayer**—boolean
* **+Undismissable**—boolean
* **+NeverPickupWeapons**—boolean
* **+NeverSwitchWeapons**—boolean
* **+AttackEnemiesOnSight**—boolean
* **+AttackPlayerOnSight**—boolean
* **+RespawnAfterDeath**—boolean
* **+Sleeping**—boolean
* **+Group**—string
* **+Variant**—string
* **+MinRank**—integer

### Examples

```
$Human:					"sr2_mp_sa_museum2_$Mission_police_1_1"
$Char type:				"npc_cop"
$Start nav:				"sr2_mp_sa_museum2_$Mission_police_1_1"
+Hit points:			500
+Item:					"flashbang"
+Item:					"stun_gun"
+Item:					"nightstick"
+Item:					"magnum"
+Team:					"Police"
+Personality:			"police normal"
+DontDropWeaponOnRagdoll
+Group:					"sr2_mp_sa_museum2_$Mission_police_npcs_1"

$Human:					"ss04_$Shaundi"
$Char type:				"npc_Shaundi"
$Start nav:				"ss04_$Shaundi"
+Personality:			"civilian cowardly"
+Cower/Flee:			"always cower when attacked"
+IgnoreAttacks
+ShouldNotBeGrabbed
+NeverTurnOnPlayer
+Undismissable
+NeverPickupWeapons
+NeverSwitchWeapons
+Group:					"ss04_$Veteran_Child_and_Shaundi"
```

## Spawn NPC Regions

This section is always empty when present.

## Racing

This section is always empty when present.

## Strongholds

This section is always empty when present.

## Action Nodes

!!! note "Format exception"
    Note that action nodes are contained in their own files, which use a slightly different file format to other CTS files. See [here](#action-node-files) for more details.

* **$Node Type**—string
* **$Action Node**—string
* **$Pos**—vec3 co-ordinates
* **$Orient**—see [above notes](#orient)
* **$Object**—string; always empty
* **$Object Chunk UID**—integer; always 0
* **$Object Pos**—vec3 co-ordinates
* **$Object Orient**—see [above notes](#orient)

### Example

```
$Node Type:		"SPAWN_Workers"
$Action Node:	"ANode_$SPAWN_Wo_c070i_0000 (0)"
$Pos:				<-555.219971 5.002846 614.405212>
$Orient:			[0.418879]
	$Object:	""
	$Object Chunk UID:	0
	$Object Pos:	<0 0 0>
	$Object Orient:	[I]
```

## Parking

* **$Parking**—string
* **$Dimensions**—float[2]
* **$Spawn Chance**—integer; appears to be scale of 0-100
* **+Force vehicle**—string
* **+AllowTrailer**—boolean
* **+HeadInOnly**—boolean
* **+ParkingGarage**—boolean

### Example

```
$Parking:			"parking_spots_$parking_020"
$Dimensions:		6.00 2.50
$Spawn Chance:		7
+Force vehicle:	"truck_2dr_construct01"
+AllowTrailer
```

## ChunkStreamingTestCases

This section is always empty when present.

## Ambients

* **$Ambient**—string
* **$Navpoint**—string; navpoint name
* **$Shape**—[Shape](#shape)
* **$Inner Offset**—vec3
* **$Attachment Type**—string; can be "None", "Effect", "Object", "Mission" or "Chunk"
* **$Chunk**—Int64
* **$Chunk Effect**—Int64[]
* **$Chunk Object**—Int64[]
* **$Mission**—string
* **$Emitter**—[Emitter](#emitter)[]
* **+Masked**—boolean

### Example

```
$Ambient:	"amb_sr2_mp_custbg_RT"
	$Navpoint:	"amb_sr2_mp_custbg_RT"
	$Shape:	"Box"
		$Inner Dimensions:	<75.625000 51 19>
		$Outer Dimensions:	<75.750000 84.875000 52.500000>
	$Inner Offset:	<0 0 0>
	$Attachment Type:	"None"
	$Emitter:	"list_0"
		$Volume:	0.1750
		$Priority:	0.7700
		$Min Delay:	0
		$Max Delay:	0
		$Min ToD:	0
		$Max ToD:	0
		$ToD Fade In:	0
		$ToD Fade Out:	0
		$Play Type:	"In Order"
		+File:	"amb_int_room_med_1"
	$Emitter:	"radio"
		$Volume:	0.5000
		$Priority:	0.0000
		$Radio:	"MP_Customization"
		$Min ToD:	0
		$Max ToD:	0
		$ToD Fade In:	0
		$ToD Fade Out:	0
	+Masked
```

## DSP Regions

* **$DSP**—string
* **$Navpoint**—string; navpoint name
* **$Shape**—[Shape](#shape)
* **$Inner Offset**—vec3
* **$Interior Reverb Send Level**—float
* **$Exterior Reverb Send Level**—float
* **+Masked**—boolean

### Example

```
$DSP:	"amb_redlight_DSP_Shanty"
	$Navpoint:	"amb_redlight_DSP_Shanty"
	$Shape:	"Box"
		$Inner Dimensions:	<213 39.750000 376.500000>
		$Outer Dimensions:	<213.125000 39.875000 376.625000>
	$Inner Offset:	<0 0 0>
	$Interior Reverb Send Level:	0.000
	$Exterior Reverb Send Level:	0.325
	+Masked
```

## Audio Cluders

* **$Cluder**—string
* **$Navpoint**—string; navpoint name
* **$Door**—boolean
* **$Chunk Object**—Int64[]
* **$Mesh Chunk UID**—Int64[]
* **$Mesh UID**—Int64[]
* **$Rotation**—vec3
* **$Inner**—float[3]
* **$Outer**—float[2]
* **$Offset**—float[2]
* **+Building Exit**—boolean
* **+On Corner**—boolean
* **+Right Side Corner**—boolean
* **+Masked**—boolean

### Example

```
$Cluder:	"amb_cluder_$Airport9"
	$Navpoint:	"amb_cluder_$Airport9"
	$Door:
	$Mesh Chunk UID:	94
	$Mesh UID:	4294967295
	$Rotation:	<0 0 0>
	$Inner:	2.625000	2.625000	0.181954
	$Outer:	3.375000	2.875000
	$Offset:	0.000000	0.000000
	+Masked
```

## Nodes

* **$Node**—string
* **$Pos**—vec3 co-ordinates
* **$Orient**—see [above notes](#orient)
* **$AngleLeft**—float
* **$AngleRight**—float
* **$EdgeDistLeft**—float
* **$EdgeDistRight**—float

### Example

```
$Node:				"docks_$cvr017"
$Pos:					<-684.455139 15.756905 1575.781128>
$Orient:				[-1.850062]
$Angle left:		1.558446
$Angle right:		0.785405
$Edge dist left:	23.141356
$Edge dist right:	1.414323
```

## Searchlights

* **$Searchlight**—string
* **$Type**—string; is always "Basic"
* **$Anim**—string
* **$Vert Min**—float
* **$Vert Max**—float
* **$Horz Min**—float
* **$Horz Max**—float
* **+TrackPlayers**—boolean
* **+TrackPlayerFollowers**—boolean
* **$Min Player Notoriety**—integer

### Example

```
$Searchlight:					"nuke_$searchlight01"
$Type:					"Basic"
$Anim:					"horizontal_sweep"
$Vert Min:					-0.88
$Vert Max:					-0.28
$Horz Min:					-1.22
$Horz Max:					1.22
+TrackPlayers
$Min Player Notoriety:			1
```

## Movers

* **$Mover**—string
* **$Object UID**—Int64
* **$Object Name**—string
* **$Chunk**—string; chunk name
* **+Hit points**—integer
* **+VerticalDoor**—boolean

### Examples

```
$Mover:		"ss03_$Chem_Depot_01"
$Object Name:	"SU128_BarrelFurtilizer01"
$Chunk:		"sr2_chunk128"
+Hit points:			2000

$Mover:		"ep02_$MMdoor_apc"
$Object UID:	46978
$Chunk:		"SR2_UG_chunk095_UB01"
+VerticalDoor
```

## Interiors

* **$Interior**—string
* **$Object UID**—Int64
* **$Chunk**—string; chunk name
* **+Police Notoriety**—integer

### Example

```
$Interior:		"shops_sr2_city_$UN_gas_int"
$Object UID:	47819
$Chunk:		"sr2_chunk118_GS"
+Police Notoriety:	0
```

## TaggingSpots

* **$Tagging Spot**—string
* **$Pos**—vec3 co-ordinates
* **$Orient**—see [above notes](#orient)
* **$Chunk**—string; chunk name
* **+Ambient**—boolean

### Example

```
$Tagging Spot:	"tagging_spots_spot00"
$Pos:		<737.151611 11.547469 -2042.812012>
$Orient:	[-1.611664]
$Chunk:		"sr2_chunk182"
+Ambient
```

## Assets

List of strings.

### Examples

```
// -------
#Assets
// -------
$Asset:		"compCam.smeshx"
$Asset:		"gml1_bs_sd_signin.animx"
$Asset:		"gfl1_bs_run_hcking.animx"
$Asset:		"gfl1_bs_walk_hcking.animx"
$Asset:		"gfl1_bs_sd_rd_hcking.animx"
$Asset:		"gfl1_bs_sd_hcking.animx"
$Asset:		"gfl1_bs_standto_hcking.animx"
```

# Subtypes

## Shape

* **$Shape**—string; can be one of "Box", "Spline Tube", "Sphere", "Spline Box" or "Cylinder"
* **$Inner Dimensions**—vec3
* **$Outer Dimensions**—vec3
* **$Inner Radius**—float
* **$Outer Radius**—float
* **$Inner Height**—float
* **$Outer Height**—float
* **$Knot**—vec3[]

### Examples

```
$Shape:	"Cylinder"
    $Inner Radius:	617.13
    $Outer Radius:	770.13
    $Inner Height:	21.50
    $Outer Height:	370.63
    
$Shape:	"Spline Tube"
    $Inner Radius:	5.88
    $Outer Radius:	57.88
    $Knot:	<-89.740372 -6.029552 1333.107544>
    $Knot:	<-66.573921 -6.029553 1369.219238>
    $Knot:	<-44.382969 -6.029553 1399.490601>
    $Knot:	<-24.631512 -6.029556 1428.242554>
    $Knot:	<-21.982393 -6.029558 1462.313843>
    $Knot:	<-38.264965 -6.029556 1484.628418>
    $Knot:	<-64.445610 -6.029557 1499.488525>
    $Knot:	<-86.900108 -6.029560 1505.799438>
    $Knot:	<-121.373367 -6.029560 1501.184204>
    $Knot:	<-161.721863 -6.029559 1479.357666>
    $Knot:	<-188.090714 -6.029554 1444.048706>

    
$Shape:	"Box"
    $Inner Dimensions:	<36.125000 63.875000 117.750000>
    $Outer Dimensions:	<40 67.375000 121.500000>
```

## Emitter

* **$Emitter**—string
* **$Volume**—float
* **$Priority**—float
* **$Min Delay**—integer
* **$Max Delay**—integer
* **$Min ToD**—integer
* **$Max ToD**—integer
* **$ToD Fade In**—integer
* **$ToD Fade Out**—integer
* **$Play Type**—string; can be "Random" or "In Order"
* **+Ambient Tone**—boolean
* **+Force Exterior Source**—boolean
* **+Silence on Weapon Fire**—boolean
* **+Silence on Rain**—boolean
* **+File**—string[]

### Example

```
$Emitter:	"gulls"
    $Volume:	0.3375
    $Priority:	0.2000
    $Min Delay:	6000
    $Max Delay:	12000
    $Min ToD:	600
    $Max ToD:	2200
    $ToD Fade In:	30000
    $ToD Fade Out:	30000
    $Play Type:	"Random"
    +Silence on Rain
    +File:	"amb_ext_seagulls_01"
    +File:	"amb_ext_seagulls_02"
    +File:	"amb_ext_seagulls_03"
    +File:	"amb_ext_seagulls_04"
    +File:	"amb_ext_seagulls_05"
    +File:	"amb_ext_seagulls_06"
    +File:	"amb_ext_seagulls_group_01"
    +File:	"amb_ext_seagulls_group_02"
    +File:	"amb_ext_seagulls_group_03"
    +File:	"amb_ext_seagulls_group_04"
    +File:	"amb_ext_seagulls_group_05"
```

# Format exceptions

## Action node files

Action node files have no section headings whatsoever, nor a ```#End``` at the file. Additionally, their name field comes second, not first. They are otherwise typical CTS files.

These are generally, but not always, files matching the regular expression ```/sr2(_[A-Z]{2})?_chunk[0-9]{3}(_[A-Z]{2})?\.cts/i```.

## st2_meshlibrary_fade_dists.cts

This file, located within the archive ```common.vpp_pc```, is effectively just a CSV. Each line has a string encased in double quotes and a floating point number, separated by two tab characters.