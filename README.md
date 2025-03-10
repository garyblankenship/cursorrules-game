# Guide to Creating CursorRules Text Adventure Games

This guide will walk you through the process of creating your own text adventure games using the CursorRules format and how the engine interprets and runs your game files.

## Table of Contents
1. [Understanding the CursorRules Format](#understanding-the-cursorrules-format)
2. [Creating Your Game File](#creating-your-game-file)
3. [Game Components](#game-components)
4. [How the Engine Processes Your Game](#how-the-engine-processes-your-game)
5. [Advanced Techniques](#advanced-techniques)
6. [Testing and Debugging](#testing-and-debugging)
7. [Example Game](#example-game)

## Understanding the CursorRules Format

A CursorRules file (`.cursorrules`) is a text file with JavaScript-like syntax that defines all the components of your text adventure game. The engine parses this file to create an interactive experience where players can navigate locations, interact with items, and solve puzzles.

The format consists of four main sections:
- State: The initial game state (inventory, flags, current location)
- Items: Definitions of all objects in the game
- Locations: Definitions of all places the player can visit
- Rules: Functions that process player commands

## Creating Your Game File

### Step 1: Set Up the Basic Structure

Start by creating a new file with the `.cursorrules` extension. Here's a skeleton structure:

```javascript
// My Adventure Game
// .cursorrules file

// Initial game state
let state = {
  inventory: [],
  visited: {},
  flags: {
    // Game-specific flags go here
  },
  currentLocation: "start"
};

// Item definitions
const items = {
  // Item definitions go here
};

// Location definitions
const locations = {
  // Location definitions go here
};

// Game rules
const rules = {
  // Rule definitions go here
};

// Export for the engine
export { state, locations, items, rules };
```

### Step 2: Save Your File

Save your file with a descriptive name, for example `haunted-mansion.cursorrules`, and place it in the `public/games` directory of your Laravel application.

## Game Components

### Defining the Initial State

The state object tracks the player's progress and contains:

```javascript
let state = {
  inventory: [],         // Items the player is carrying
  visited: {},           // Locations the player has seen
  flags: {               // Game-specific flags
    doorUnlocked: false,
    lampLit: false,
    // Add your own flags here
  },
  currentLocation: "entrance"  // Starting location ID
};
```

### Defining Items

Items are objects the player can interact with. Each item has properties:

```javascript
const items = {
  key: {
    name: "brass key",
    description: "A small brass key with intricate engravings.",
    portable: true       // Can the player pick it up?
  },
  book: {
    name: "ancient tome",
    description: "A dusty book with strange symbols on the cover.",
    portable: true,
    read: false          // Custom property for game logic
  },
  door: {
    name: "wooden door",
    description: "A heavy wooden door with a brass lock.",
    portable: false      // Fixed objects should be non-portable
  }
};
```

### Defining Locations

Locations are the places the player can visit:

```javascript
const locations = {
  entrance: {
    name: "Mansion Entrance",
    description: "You stand before an imposing mansion. Rain pours down around you, and the door is slightly ajar.",
    items: ["key"],      // Items present in this location
    exits: {
      north: "hallway",  // Direction: destination location ID
      south: "garden"
    },
    firstVisit: true     // Special flag for first-time visit text
  },
  hallway: {
    name: "Grand Hallway",
    description: "A dusty hallway stretches before you. Portraits of stern-looking ancestors line the walls.",
    items: ["book"],
    exits: {
      south: "entrance",
      east: "library",
      west: "diningRoom"
    },
    firstVisit: true
  }
};
```

### Defining Rules

Rules are functions that process player commands:

```javascript
const rules = {
  // Movement rule
  movement: (input) => {
    const direction = input.toLowerCase();
    if (["north", "south", "east", "west"].includes(direction)) {
      const currentLocation = locations[state.currentLocation];
      
      // Check if this exit exists
      if (currentLocation.exits[direction]) {
        const destinationId = currentLocation.exits[direction];
        state.visited[state.currentLocation] = true;
        state.currentLocation = destinationId;
        
        // Return location description
        return `${locations[destinationId].name}\n${locations[destinationId].description}`;
      } else {
        return "You can't go that way.";
      }
    }
    return null;  // Null means the rule didn't match
  },
  
  // Look rule
  look: (input) => {
    if (input.match(/^(look|look around|l)$/i)) {
      const currentLocation = locations[state.currentLocation];
      return `${currentLocation.name}\n${currentLocation.description}`;
    }
    return null;
  },
  
  // Custom rule example - unlock door
  unlockDoor: (input) => {
    if (input.match(/^(unlock door|use key on door)$/i)) {
      if (state.currentLocation === "entrance") {
        if (state.inventory.includes("key")) {
          state.flags.doorUnlocked = true;
          return "You unlock the door with the brass key. It swings open with a creak.";
        } else {
          return "You need a key to unlock this door.";
        }
      } else {
        return "There's no door to unlock here.";
      }
    }
    return null;
  }
};
```

## How the Engine Processes Your Game

When a player loads your game, here's what happens:

1. **Initialization**:
   - The engine loads and parses your `.cursorrules` file
   - It converts JavaScript syntax to PHP data structures
   - It initializes the game state based on your `state` object

2. **Command Processing**:
   - When the player enters a command, the engine passes it to each rule in order
   - The first rule that returns a non-null response "wins"
   - If no rules match, the engine returns a default message

3. **State Management**:
   - The engine updates the game state based on rule actions
   - It saves the state between commands (using the session or cache)
   - It tracks the command history and provides it to the UI

4. **Response Rendering**:
   - The engine formats the response from the matched rule
   - It adds additional information like available exits and visible items
   - It sends the formatted response back to the player

## Advanced Techniques

### Conditional Exits

You can make exits conditional based on game state:

```javascript
exits: {
  north: "treasury",
  east: {
    locationId: "secretRoom",
    condition: () => state.flags.foundSecretDoor
  }
}
```

In this case, the east exit will only appear if the `foundSecretDoor` flag is true.

### Custom Item Interactions

Create specific rules for item interactions:

```javascript
readBook: (input) => {
  if (input.match(/^(read book|read tome|open book)$/i)) {
    if (state.inventory.includes("book")) {
      items.book.read = true;
      state.flags.learnedSpell = true;
      return "You read the ancient tome and learn a powerful spell. The words seem to glow on the page.";
    } else if (locations[state.currentLocation].items.includes("book")) {
      return "You should pick up the book first.";
    } else {
      return "You don't see any book here.";
    }
  }
  return null;
}
```

### Event Triggers

Create events that trigger based on player actions:

```javascript
checkGhostAppearance: (input) => {
  // This rule checks after every command
  if (state.currentLocation === "masterBedroom" && 
      state.flags.midnightHour && 
      !state.flags.ghostAppeared) {
    state.flags.ghostAppeared = true;
    return "As you enter, the temperature drops suddenly. A ghostly figure materializes before you!";
  }
  return null;
}
```

## Testing and Debugging

When creating your game, test it thoroughly:

1. **Check All Locations**:
   - Make sure every location can be reached
   - Verify that all exits lead to the correct destinations

2. **Test Item Interactions**:
   - Try picking up and dropping all items
   - Test all special interactions (using, combining, etc.)

3. **Verify Game Logic**:
   - Test all puzzles and their solutions
   - Make sure flags are set and checked correctly

4. **Edge Cases**:
   - Try unusual or unexpected commands
   - Test boundary conditions (inventory limits, etc.)

If you encounter issues, check the Laravel log files for error messages and details.

## Example Game

Below is a simple example game to help you get started:

```javascript
// Enchanted Forest Adventure
// .cursorrules file

// Initial game state
let state = {
  inventory: [],
  visited: {},
  flags: {
    talkedToElf: false,
    foundMagicStone: false,
    openedPortal: false
  },
  currentLocation: "forestEdge"
};

// Item definitions
const items = {
  berry: {
    name: "glowing berry",
    description: "A small berry that emits a soft blue light.",
    portable: true
  },
  stone: {
    name: "ancient stone",
    description: "A smooth stone covered in mysterious runes.",
    portable: true
  },
  staff: {
    name: "wooden staff",
    description: "A gnarled wooden staff that feels warm to the touch.",
    portable: true
  }
};

// Location definitions
const locations = {
  forestEdge: {
    name: "Forest Edge",
    description: "You stand at the edge of an ancient forest. Towering trees loom ahead, their branches swaying gently in the breeze. A narrow path leads deeper into the woods.",
    items: ["berry"],
    exits: {
      north: "deepWoods",
      east: "meadow"
    },
    firstVisit: true
  },
  deepWoods: {
    name: "Deep Woods",
    description: "The forest grows denser here, with sunlight barely penetrating the thick canopy. Strange sounds echo from all directions.",
    items: [],
    exits: {
      south: "forestEdge",
      west: "clearing"
    },
    firstVisit: true
  },
  clearing: {
    name: "Forest Clearing",
    description: "A peaceful clearing bathed in gentle sunlight. A small stream trickles through the center, and wildflowers dot the ground.",
    items: ["stone"],
    exits: {
      east: "deepWoods",
      north: "elfTree"
    },
    firstVisit: true
  },
  meadow: {
    name: "Sunlit Meadow",
    description: "A wide meadow filled with swaying grass and colorful flowers. Butterflies dance in the gentle breeze.",
    items: [],
    exits: {
      west: "forestEdge",
      north: "ancientOak"
    },
    firstVisit: true
  },
  elfTree: {
    name: "Elven Tree Home",
    description: "A massive tree with a dwelling built into its trunk. Intricate carvings adorn the wooden walls.",
    items: ["staff"],
    exits: {
      south: "clearing"
    },
    firstVisit: true
  },
  ancientOak: {
    name: "Ancient Oak",
    description: "A towering oak tree that radiates an aura of ancient magic. At its base is a closed portal carved into the trunk.",
    items: [],
    exits: {
      south: "meadow",
      portal: "faeRealm"
    },
    firstVisit: true
  },
  faeRealm: {
    name: "Fae Realm",
    description: "A shimmering otherworldly landscape with floating islands and crystal formations. The air seems to sparkle with magic.",
    items: [],
    exits: {
      portal: "ancientOak"
    },
    firstVisit: true
  }
};

// Game rules
const rules = {
  // Standard movement rule
  movement: (input) => {
    const direction = input.toLowerCase();
    if (["north", "south", "east", "west", "portal"].includes(direction)) {
      const currentLocation = locations[state.currentLocation];
      
      // Check if this exit exists
      if (currentLocation.exits[direction]) {
        // Special case for portal
        if (direction === "portal" && !state.flags.openedPortal) {
          return "The portal is currently closed. Perhaps there's a way to open it?";
        }
        
        const destinationId = currentLocation.exits[direction];
        state.visited[state.currentLocation] = true;
        state.currentLocation = destinationId;
        
        const destination = locations[destinationId];
        let response = `${destination.name}\n${destination.description}`;
        
        // Add items description
        if (destination.items.length > 0) {
          response += "\n\nYou see:";
          destination.items.forEach(itemId => {
            response += `\n- ${items[itemId].name}`;
          });
        }
        
        // Add exits description
        response += "\n\nExits:";
        Object.keys(destination.exits).forEach(dir => {
          if (dir === "portal" && !state.flags.openedPortal) return;
          response += ` ${dir}`;
        });
        
        return response;
      } else {
        return "You can't go that way.";
      }
    }
    return null;
  },
  
  // Look around rule
  look: (input) => {
    if (input.match(/^(look|look around|l)$/i)) {
      const currentLocation = locations[state.currentLocation];
      
      let response = `${currentLocation.name}\n${currentLocation.description}`;
      
      // Add items description
      if (currentLocation.items.length > 0) {
        response += "\n\nYou see:";
        currentLocation.items.forEach(itemId => {
          response += `\n- ${items[itemId].name}`;
        });
      }
      
      // Add exits description
      response += "\n\nExits:";
      Object.keys(currentLocation.exits).forEach(dir => {
        if (dir === "portal" && !state.flags.openedPortal) return;
        response += ` ${dir}`;
      });
      
      return response;
    }
    return null;
  },
  
  // Inventory rule
  inventory: (input) => {
    if (input.match(/^(inventory|i|inv)$/i)) {
      if (state.inventory.length === 0) {
        return "You aren't carrying anything.";
      }
      
      let response = "You are carrying:";
      state.inventory.forEach(itemId => {
        response += `\n- ${items[itemId].name}`;
      });
      
      return response;
    }
    return null;
  },
  
  // Take item rule
  take: (input) => {
    const match = input.match(/^(take|get|pick up)\s+(.+)$/i);
    if (match) {
      const itemName = match[2].toLowerCase();
      const currentLocation = locations[state.currentLocation];
      
      // Find item in current location
      const itemId = currentLocation.items.find(id => 
        items[id].name.toLowerCase().includes(itemName));
      
      if (itemId) {
        const item = items[itemId];
        
        if (item.portable) {
          // Remove from location and add to inventory
          currentLocation.items = currentLocation.items.filter(id => id !== itemId);
          state.inventory.push(itemId);
          
          // Special case for magic stone
          if (itemId === "stone") {
            state.flags.foundMagicStone = true;
          }
          
          return `You take the ${item.name}.`;
        } else {
          return `You can't take the ${item.name}.`;
        }
      } else {
        return "You don't see that here.";
      }
    }
    return null;
  },
  
  // Talk to elf rule
  talkToElf: (input) => {
    if (input.match(/^(talk|speak|talk to elf|speak to elf)$/i) && 
        state.currentLocation === "elfTree") {
      if (state.flags.talkedToElf) {
        return "The elf smiles at you. 'Remember, the staff and stone together will open the portal.'";
      } else {
        state.flags.talkedToElf = true;
        return "A slender elf emerges from the tree dwelling. 'Welcome, traveler,' they say. 'I've been expecting you. The ancient oak holds a portal to the Fae Realm, but it needs magic to open. Find the ancient stone and use my staff to activate it.'";
      }
    }
    return null;
  },
  
  // Open portal rule
  openPortal: (input) => {
    if (input.match(/^(open portal|use staff|use stone|use staff and stone)$/i) && 
        state.currentLocation === "ancientOak") {
      if (!state.inventory.includes("staff")) {
        return "You need a magical focus to channel energy toward the portal.";
      }
      
      if (!state.inventory.includes("stone")) {
        return "You need a source of magical energy to open the portal.";
      }
      
      state.flags.openedPortal = true;
      return "You hold the staff in one hand and the stone in the other. Energy crackles between them, then shoots toward the tree trunk. The carved portal begins to glow, revealing a shimmering gateway to another realm.";
    }
    return null;
  },
  
  // Help rule
  help: (input) => {
    if (input.match(/^(help|h|\?)$/i)) {
      return `Commands:
- look: Examine your surroundings
- inventory (or i): Check what you're carrying
- take [item]: Pick up an item
- drop [item]: Put down an item
- examine [item]: Look closely at an item
- north, south, east, west: Move in that direction
- talk: Speak with someone if present
- use [item]: Use an item in some way
- help (or ?): Show this help text`;
    }
    return null;
  }
};

// Export for the engine
export { state, locations, items, rules };
```

Place this file in your `public/games` directory as `enchanted-forest.cursorrules` and you'll have a complete playable adventure!

## Conclusion

Creating your own text adventures with CursorRules is a rewarding way to craft interactive stories. By defining locations, items, and rules, you can build complex worlds for players to explore. Start with a simple game and gradually add more features as you become comfortable with the format.

Remember that the most important aspect of a text adventure is the quality of the writing and the coherence of the game world. Focus on creating interesting descriptions, logical puzzles, and a compelling narrative to keep players engaged.

Happy game creating!
