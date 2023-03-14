---
title: Notes on Using and Extending Unity's Tilemap
draft: true

---

## Tilemap Component

The basic goal is to create some kind of map manager, so we can put elements on a grid, check what objects are where, do pathfinding, and so on. Without getting any more specific than that (which would be just programming the thing, just go look at my source code), here are some takeaways and lessons that I have learned.

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