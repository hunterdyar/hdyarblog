---
title: Notes on Using and Extending Unity's Tilemap
draft: true

---

## Tilemap Component

The basic goal is to create some kind of map manager, so we can put elements on a grid, check what objects are where, do pathfinding, and so on. Without getting any more specific than that (which would be just programming the thing, just go look at my source code), here are some takeaways and lessons that I have learned writing various tools and projects that use the tilemap.

### Tilemap Good Practices

- Organize the palette to be easy to draw, arrange tiles together sensibly.
- Take advantage of scriptable tiles to minimize tile assets and tedious work.
  - In many cases, you might want to just use the Rule Tile found in [2D Tilemap Extras](https://docs.unity.cn/Packages/com.unity.2d.tilemap.extras@1.8/manual/Tiles.html), avoiding having to write any of these things from scratch.

## Extending The Tilemap

### Interfacing with Tilemap

Tilemap is a sealed class, which means we can't extend it and make our own version of it. (Why is it sealed? A Unity Forum post [from 2008](https://forum.unity.com/threads/sealed-classes-in-u3.60638/#post-440101) shed's light on the design decision: It would bork with deserialization, and means unity can focus on a public api they can keep documented and stable, separate from whatever is going on in their internal/private api.

So, we can't - and, arguably shouldn't - make our own Tilemap. I like to make my own script on the same game object.

Tilemap provides an event for when tiles are updated, but it's editor only.

### Read Or Clone Tile Data

Tilemap isn't a convenient solution for operating with. When we poll what Tile is at a location, it gives us a reference to the Tile scriptableObject. If i want to read "is this grass or dirt?", that's fine, but writing to it would modifiy every tile of that type. Probably not what we want for, say, storing game logic, like player positions. Instead, I prefer to create my own Dictionary that has a key of type Vector3Int - the tile position, and a value of some node class. The node class can store a reference to what type of tile is at this location, we can use it for pathfinding, it can cache a reference to it's neighbors, be updated at runtime, connect to other nodes in ways the tilemap doesn't intend (off-map links? doors? teleports?) and more. 

What's more, when we create the nodes, we can convert from the grid space that Unity provides to our own. Perhaps going from Vector3Int to Vector2Int to drop an uneeded z value, or - see below about hex grids - converting into a more convenient coordinate system for algorithms to run on.

The downside is that we have to keep our node network in sync with the tilemap. Most projects I work on don't update the tilemap at runtime, but for us, we would have to re-scan the tilemap and recreate or merge our node maps after any changes were made, an expensive process.

### Getting All Tiles

Code snippet for looping through the data.

## Agents On The Map

Having a map is good, but we want to know when where everything is. Is there a block next to the player? Is there an enemy under this tile? Can I move in this direction? 

> I like to call anything that can go on a tile, move between tiles, and not occupy the same space as another agent, an "agent". 

The decision to make is: Should agents keep track of what NavTile they are on, or should NavTiles keep track of what agents are on them? What you need to write convenient and easy to check code is **both**. I want to ask agents what tiles they are on, find that tiles neighbors, and ask those tiles if they have agents. This should be possible and easy. But, if I am storing a single relationship in two places, it can easily get our of sync. What I want to avoid to prevent a bug-nightmare is **both**. How do we compromise?

The trick is not deciding where to store the data - both places can have read-only references to their current tile/agents. The trick is deciding on a single source of truth to update it, and have that update the values appropriately. We make it impossible to update tiles *without* using our special function that handles the interdependency, and we should be good enough. 

### Options

1. Agents move onto tiles.
2. Tiles accept Agents.
3. Use the Tilemap.

I like option 3 for small projects. Agent's basically have to have a reference to the tilemap anyway to figure out where they are standing on start, even for simple agents that may never move. You don't have to work that way. I've seen many systems that put colliders on all the tiles, and have agents raycast down to determine where they are, and update themselves through the tiles. I don't like that solution, but if one is always consistent with how the tiles get updated, it will work. 

The downside of having the Tilemap hold onto "MoveAgentToTile" functions is that it gets rather complex - that's a lot of logic and systems for a single Unity component to handle and interact with. It gets particularly challenging if we want to support multiple maps, like dynamically loading in scenes (new rooms) and having them connect. If an agent just talks to it's tile, that complexity of when it "switches maps" can be hidden from the agent, and we don't need to worry about updating some "current tilemap" reference. That's why I only like option 3 for small projects, where each level is just a pre-determined room with no runtime loading or unloading, of off-map links. I prefer Option 2, where the Nodes contain the interfaces for updating the map, otherwise. 

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