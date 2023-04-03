---
title: Notes on Using and Extending Unity's Tilemap
draft: true

---

## Tilemap Component

Unity's tilemap is great, but it's primarily a design tool, not really focused on providing runtime or game-logic solutions. It would be great if we could continue to design with the tilemap as game designers, and have the game just work from a systems point of view. So, we want to be able to inspect the grid, check what objects are where, do pathfinding, and so on. 

Without getting any more specific than that (which would be just programming the thing, just go look at my source code), here are some takeaways and lessons that I have learned writing various tools and projects that use Unity's tilemap.

### Tilemap Good Practices

- Organize the palette to be easy to draw, arrange tiles together sensibly.
- Take advantage of scriptable tiles to minimize tile assets and tedious work.
  - In many cases, you might want to just use the Rule Tile found in [2D Tilemap Extras](https://docs.unity.cn/Packages/com.unity.2d.tilemap.extras@1.8/manual/Tiles.html), avoiding having to write any of these things from scratch.
- Use multiple Tilemaps inside of the same grid to handle, for example, separate colliders/triggers.

## Extending The Tilemap

### Interfacing with Tilemap

Tilemap is a sealed class, which means we can't extend it and make our own version of it. (Why is it sealed? A Unity Forum post [from 2008](https://forum.unity.com/threads/sealed-classes-in-u3.60638/#post-440101) shed's light on the design decision: It would bork with deserialization, and means unity can focus on a public api they can keep documented and stable, separate from whatever is going on in their internal/private api.

So, we can't - and, arguably shouldn't - make our own Tilemap. Instead, we have to talk to the tilemap with our own script. Good ol' fashioned GetComponent.

Tilemap provides an event for when tiles are updated, but it's editor only.

### Read Or Clone Tile Data

Tilemap isn't a convenient solution for operating with. When we poll what Tile is at a location, it gives us a reference to the Tile scriptableObject. If i want to read "is this grass or dirt?", that's fine, but updating TileData would modify every tile of that type, and would modify editor-time scriptableObject data. Probably not what we want for game logic. 

Instead, I prefer to create my own Dictionary that has a key of type Vector3Int - the tile position, and a value of some node class I create (a POCO). The node class can store a reference to what type of tile is at this location, we can use it for pathfinding, it can cache a reference to it's neighbors, be updated at runtime, connect to other nodes in ways the tilemap doesn't intend (off-map links? doors? teleports?) and more. 

What's more, when we create the nodes, we can convert from the grid space that Unity provides to our own coordinate system. Perhaps going from Vector3Int to Vector2Int to drop an uneeded z value, or - see below about hex grids - converting into a more convenient coordinate system for algorithms to run on.

The downside is that we have to keep our node network in sync with the tilemap. Most projects I work on don't update the tilemap at runtime, but for us, we would have to re-scan the tilemap and recreate or merge our node maps after any changes were made, an expensive process.

The upside is for things that do update at runtime - like a door opening and closing - we can update a walkable flag in our class, and do pathfinding with our class, and have the game react without *having* to update the Tile data. If we're animating things, like a door opening, we should probably just use a MonoBehaviour anyway. It opens and closes, and updates a walkable flag on the node its 'on'. 

Getting All Tiles

Code snippet for looping through the data.

## Agents On The Map

Having a map is good, but we want to know when where everything is. Is there a block next to the player? Is there an enemy under this tile? Can I move in this direction? 

> I like to call anything that can go on a tile an "Entity". Highlight tiles, status effects, special destinations, and so on. I like to call an entity that can go on a tile, move between tiles, and not occupy the same space as another, an "agent". 

The decision to make is: Should agents keep track of what NavTile they are on, or should NavTiles keep track of what agents are on them? What you need to write convenient and easy to check code is **both**. I want to ask agents what tiles they are on, find that tiles neighbors, and ask those tiles if they have agents. This should be possible and easy. But, if I am storing a single relationship in two places, it can easily get out of sync. We really want to avoid both, it's a bug-filled nightmare where we have to remember to update a single (conceptual) piece of information in two (literal) variables. How do we compromise?

The trick is not deciding where to store the data - both agents and tiles can have read-only references to their respective tile/agents. That's useful. The trick is deciding on a **single source of truth** to update it, and have that update the values appropriately. We make it impossible to update tiles *without* using our special function that handles the interdependency, and that should be good enough. 

We have multiple approaches: 

1. Store the data twice, and use a wrapper function to ensure both variables get updated.
2. Store the variable once, and use a convenience function or auto-body property to get the other seamlessly.
3. Store the information in a unique third location, like a separate dictionary of nodes and agents, and just talk to that.

The first is simpler to implement, the second relies on some references being in place, probably to the tilemap. Since agents and tiles probably need that reference anyway, this is fine. The third makes working with different kinds of entities more difficult, but allows for nodes to be completely ignorant of agents, which improves flexibility for adding new kinds of weird agents and logic in the future.

### Who Controls The Board?

1. Agents move onto tiles.
2. Tiles accept Agents.
3. Tilemap controls both as a manager.
4. Another "neutral third party" somewhere else.

I like option 3 for small projects. Agent's basically have to have a reference to the tilemap anyway to figure out where they are standing on start, even for simple agents that may never move. You don't have to have this reference, however. I've seen many systems that put colliders on all the tiles, and have agents raycast down to determine where they are, and update themselves through the tiles. There is a certain simplicity in separating the "map stuff" from "player stuff". I don't like that solution, but if one is always consistent with how the tiles get updated, it will work. 

> To keep 'agent stuff' and 'map' stuff separate, I prefer to separate components. The player controls player logic, and tells it's agent component when to move, and so on. You can also have player logic extend your agent class, whatever seems simpler to work with for you probably is.

The downside of having the Tilemap hold onto "MoveAgentToTile" functions is that it gets rather complex - that's a lot of logic and systems for a single Unity component to handle and interact with. It gets particularly challenging if we want to support multiple maps, like dynamically loading in scenes (new rooms) and having them connect. If an agent just talks to it's tile, that complexity of when it "switches maps" can be hidden from the agent, and we don't need to worry about updating some "current tilemap" reference. That's why I only like option 3 for small projects, where each level is just a pre-determined room with no runtime loading or unloading, of off-map links. I prefer Option 2, where the Nodes contain the interfaces for updating the map, otherwise. 

### A ScriptableObject Solution for Agents & Map Layers

Let's back up and consider what type of API we might need the agents to have. Agents will need to poll a tile and discover everything that is there: what enemies, status effects, terrain types, and so on. Imagine a tactics game, a tank finishes a move, and has to determine effects it needs to now apply (like "on fire") or ("activate this mine I stepped on"). Although, it probably doesn't need all of that at once, it's different systems are probably concerned with different things-on-the-tile. First do movement costs, then health looks at statuses, then inventory looks at pickups, etc.

For many minds of games, it's probably not that often that we actually need to search for multiple *types* of entities at once. And, usually, while tiles can often have multiple *things* on it: terrain, potion, and arriving wizard character - it doesn't often have multiple of *one type of thing*. 

> Instead of looping through a list of things of all types, looking for the ones of the type I care about, we can store our things by type - not store them in lists by tile.

For any conceptual layer, one can create an instance of a ScriptableObject (I call mine "Entity Maps") that are glorified wrappers for a dictionary* of Nodes and Entities. We make one for vehicles, one for status effects, one for special terrain types, one for pickup items, one for the highlighted squares under the mouse, one for valid destinations to move to, and so on. Any given system can look at the specific one it cares about. Should we need to actually get *all* of the entities on one node, we can loop through them appropriately.  

> *I used a bi-directional dictionary pattern. Actually, I used two dictionaries (key-> value and an inverse value-> key), since C# 8 doesn't have a built-in bidirectional dictionary, and I didn't feel like writing a wrapper data structure for one, when my scriptable object is already a wrapper data structure.

But what about when we DO want multiple items on one layer and in one entitymap? For my solution, one could wrap the list of entities, such as "Status Effects" up in a single Entity gameobject that manages it's various status effects visuals (as children), which do not have to be "grid entities", but are just some child of a parent "status effect entity" that manages them. I feel this is a reasonable solution, especially because "Fire and Acid Rain" might need a different particle system than just having both the "Fire" and "Rain" particle systems turned on, for example.

What about pathfinding? Some agents should block walkability? I have no elegant solution for this, but with dictionaries, lookups should be fast. We need to ask each layer that blocks walkability if it has any agents on a tile. If only some of the entities on a layer affect walkability, then we need to get the entity and add a blocks-walking property to that agent, which would be slightly slower.

## Using Multiple Tilemaps On One Grid

The biggest reason to use multiple tilemaps is colliders.

## Hex Grids

### How The Grid Component Treats Hex Grids

Unity Uses odd offset coordinates for hex grids in it's Grid component. This allows it to store hex grid positions using only two coordinates, and presumably permits other internal optimizations that are designed around rectangular grids. Basically, it treats the hex grid as a rectangular grid, with nudging to put things into order visually. 

> Offset components store columns and rows, and then pushes either every [odd or even] [row or column] a bit for the grid to line up evenly.  

The Grid component uses odd row (odd-r) offset coordinates for pointy-top grids, and odd column (odd-q) offset coordinates for flat-top grids.

Unity doesn't store "pointy top" or "flat top" anywhere - although you will see these options in how the editor exposes features, like in the new tilemap menu. Instead, it uses the Cell Swizzle of the grid to store this. XYZ is pointy top and YXZ is flat-top, on otherwise default 2D projects.

### Working with Hex Coordinates.

I prefer to convert the unity grids into a [Cube coordinate system](https://www.redblobgames.com/grids/hexagons/#coordinates-cube) for navigation. The benefits are listed below, but first some sample code.

GridPos is how one might refer to the coordinates coming from Unity's 'grid' component, and "CubePos" refers to the hex coordinates converted to a cube coordinate system. This is just an example, I would change these to be whatever is appropriate.

This function belongs somewhere that has a reference to the **Tilemap** component, stored as "_tilemap".

```c#
public Vector3Int GridPosToCubePos(Vector3Int location)
{
			if (_tilemap.cellLayout == GridLayout.CellLayout.Hexagon)
			{
				if (_tilemap.cellSwizzle == GridLayout.CellSwizzle.XYZ)
				{
					//Unity is using Pointy Top, which uses offset odd-row coords. We will use Cube coordinates.
					return HexUtility.CubeToOddR(location);
				}
				else if (_tilemap.cellSwizzle == GridLayout.CellSwizzle.YXZ)
				{
					//Unity is using Flat Top, which uses offset odd-col coords. Again, we will use cube for both cases.
					return HexUtility.CubeToOddQ(location);
				}
			}

			return location;
		}
```

And then the Hex Utility script, referenced above, would be a static HexUtility.cs class full of many useful functions, including the Cube/Offset conversion functions. For where these functions came from, see the [Red Blob Games write-up](https://www.redblobgames.com/grids/hexagons/).

```c#
public static Vector3Int CubeToOddR(Vector3Int cube)
		{
			if (cube.x + cube.y + cube.z != 0)
			{
				Debug.LogWarning("Invalid Cube Coordinate Given");
			}
			var col = cube.x + (cube.y - (cube.y & 1)) / 2;
			var row = cube.y;
			return new Vector3Int(col, row, 0);
		}

		public static Vector3Int OddRToCube(Vector3Int hex)
		{
			var q = hex.x - (hex.y - (hex.y & 1)) / 2;
			var r = hex.y;
			return new Vector3Int(q, r, -q - r);
		}
public static Vector3Int CubeToOddQ(Vector3Int cubePos)
		{
			var col = cubePos.x;
			var row = cubePos.y + (cubePos.x - (cubePos.x & 1)) / 2;
			return new Vector3Int(col, row,0);
		}
		public static Vector3Int OddQToCube(Vector3Int pos)
		{
			var q = pos.x;
			var r = pos.y - (pos.x - (pos.x & 1)) / 2;
			return new Vector3Int(q, r, -q - r);
		}
```

### Benefits of Cube Coordinates

Cube coordinates may take 3 integers instead of 2, but they come with a lot of benefits. Most of which can be summarized as "We can often treat them like vector positions".

- We can add/subtract coordinates together like vectors
  - This helps with lots of algorithms, like finding lines, shapes, distances, and so on.
- We can scale coordinates like vectors
- We can calculate distance easily
- We can normalize and compare relative directions easily
- We can find neighbors by using a consistent list of directions (in offset we would have to check if we were on an even or odd position)
- We can change from flat-top to pointy-top without re-writing the code, because were already converting to cube coordinates.