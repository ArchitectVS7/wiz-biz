# Sprite Mapping Report - Wiz-Biz Game

## Overview
This document traces the complete connection chain from sprite files in `resources/sprites/` 
to their final on-screen usage in the game.

## Connection Chain Architecture

### 1. Sprite File → Scene Resource → Node → Script Reference → Display

**Example: Player Ship (ship3 → wizard_ice)**
```
resources/sprites/WIZ/wizard_ice/attack_1.png
	↓ [ext_resource in .tscn]
scenes/player/player.tscn (ExtResource("4_t853r"))
	↓ [node definition]
[node name="Ship" type="Sprite2D" parent="."] texture = ExtResource("4_t853r")
	↓ [script reference]
@onready var ship: Sprite2D = $Ship  ## In player.gd
	↓ [display usage]
ship.visible = false  ## When player dies
```

## Complete Sprite Usage Map

### 🏰 **WIZARD SPRITES** (Ship Replacements)

#### **WIZ/wizard/** → Basic Enemy (ship1 replacement)
- **File**: `resources/sprites/WIZ/wizard/attack_1.png` 
- **Scene**: `scenes/enemies/basic_enemy/basic_enemy.tscn`
- **Node Path**: `$Sprite2D` and `$SpriteGlow`
- **Script Reference**: `@onready var sprite: Sprite2D = $Sprite2D`
- **Usage**: Main enemy visual, ship explosion animations, hit effects
- **Animation**: Used for exhaust effects (normal/turbo variants)

#### **WIZ/wizard_fire/** → Kamikaze Enemy (ship2 replacement)  
- **File**: `resources/sprites/WIZ/wizard_fire/attack_1.png`
- **Scene**: `scenes/enemies/kamikaze/kamikaze.tscn`
- **Node Path**: `$Sprite2D` and `$SpriteGlow`
- **Script Reference**: `@onready var sprite: Sprite2D = $Sprite2D` (inherited from BasicEnemy)
- **Usage**: Kamikaze enemy visual, acceleration effects, energy type coloring
- **Special**: Changes to turbo mode when charging at player

#### **WIZ/wizard_ice/** → Player (ship3 replacement)
- **File**: `resources/sprites/WIZ/wizard_ice/attack_1.png`
- **Scene**: `scenes/player/player.tscn`
- **Node Path**: `$Ship` and `$ShipGlow`
- **Script Reference**: `@onready var ship: Sprite2D = $Ship`
- **Usage**: Player character, energy glow effects, death animations
- **Special**: Ship glow modulated by energy type (blue/green)

#### **ship6** → Boss (NOT YET CONVERTED)
- **File**: `resources/sprites/ship6/ship6.png`
- **Scene**: `scenes/enemies/boss/boss.tscn`
- **Node Path**: `$Sprite2D` and `$SpriteGlow`
- **Script Reference**: `@onready var ship: Sprite2D = $Sprite2D` (boss.gd line 43)
- **Usage**: Boss enemy visual, energy switching effects
- **Status**: ⚠️ PENDING CONVERSION TO WIZARD SPRITE

---

### 🚀 **PROJECTILE SPRITES**

#### **shot3/** → Player Projectiles
- **Files**: `shot3_1.png`, `shot3_2.png`, `shot3_3.png`, `shot3_asset.png`, `shot3_exp1-4.png`
- **Scene**: `scenes/player/shot/player_shot.tscn`
- **Node Paths**: 
  - `$Sprite2D` → Main projectile visual
  - `$ShotExplosion` → AnimatedSprite2D with explosion frames
- **Script Reference**: `@onready var sprite: Sprite2D = $Sprite2D`
- **Usage**: Player shots, hit explosions, energy type coloring
- **Animation Chain**: `shot3_asset.png` → explosion sequence → destruction

#### **shot5/** → Enemy Projectiles  
- **Files**: `shot5_asset.png`, `shot5_exp1-8.png`
- **Scene**: `scenes/enemies/shot/enemy_shot.tscn`
- **Node Paths**: 
  - `$Sprite2D` → Main projectile visual  
  - `$ShotExplosion` → AnimatedSprite2D with explosion frames
- **Script Reference**: `@onready var sprite: Sprite2D = $Sprite2D`
- **Usage**: Enemy shots, directional firing, hit explosions

---

### 💥 **EXPLOSION SPRITES**

#### **explosion1/** → Universal Explosions
- **Files**: `explosion1_1.png` through `explosion1_11.png` (11 frames)
- **Used In**: ALL character scenes (player, enemies, boss)
- **Node Paths**: `$ShipExplosion` (AnimatedSprite2D)
- **Script References**: `@onready var ship_explosion: AnimatedSprite2D = $ShipExplosion`
- **Usage**: Death animations for all characters
- **Animation**: 11-frame explosion sequence → entity destruction

---

### 🛡️ **UI SPRITES**

#### **shield/** → Player Health UI
- **Files**: `shield.png`, `shield_bar.png`
- **Scene**: `scenes/ui/ui.tscn`
- **Usage**: Health/shield indicators in game UI
- **Node Paths**: UI elements in main game interface

#### **inputs/** → Controls Menu
- **Files**: `tilemap.png`
- **Scene**: `scenes/menu/controls_menu/controls_menu.tscn`
- **Usage**: Input visualization in controls menu

---

## Script-to-Sprite Connection Details

### **Player (player.gd)**
```gdscript
@onready var ship: Sprite2D = $Ship                    # Main wizard sprite
@onready var ship_glow: Sprite2D = $ShipGlow          # Glow effect
@onready var ship_explosion: AnimatedSprite2D = $ShipExplosion  # Death animation
@onready var shot_out_effect: AnimatedSprite2D = $ShotOutEffect # Shot effects
```

### **Basic Enemy (basic_enemy.gd)**
```gdscript
@onready var sprite: Sprite2D = $Sprite2D             # Main wizard sprite
@onready var sprite_glow: Sprite2D = $SpriteGlow      # Glow effect  
@onready var ship_explosion: AnimatedSprite2D = $ShipExplosion  # Death animation
@onready var exhaust: AnimatedSprite2D = $Exhaust     # Movement effects
```

### **Boss (boss.gd)**
```gdscript
@onready var ship: Sprite2D = $Sprite2D               # Main sprite (still ship6)
# Note: Uses same pattern as BasicEnemy but different variable name
```

## Animation Systems

### **Exhaust Animations** (Movement Effects)
- **Normal Mode**: 4-frame cycle at 5 FPS
- **Turbo Mode**: 4-frame cycle at 5 FPS  
- **Used By**: All ships/wizards for movement indication
- **Current Status**: Still using ship exhaust sprites (needs wizard spell effects)

### **Shot Animations** (Projectile Effects)
- **Player Shots**: 3-frame shot cycle, 4-frame explosion
- **Enemy Shots**: 8-frame explosion sequence
- **Trigger**: On collision or screen exit

### **Explosion Animations** (Death Effects)
- **Duration**: 11 frames at 10 FPS 
- **Trigger**: When character health reaches 0
- **Cleanup**: Entity destroyed after animation completes

## Energy System Integration

### **Energy Types** (Game.EnergyType)
- **BLUE**: Default player energy
- **GREEN**: Alternate player energy  
- **Usage**: Modulates sprite colors, affects damage calculations

### **Color Modulation Chain**
```gdscript
# Example from player.gd
ship_glow.modulate = Game.ENERGY_TYPE_COLOR[_energy]
ship_glow.modulate.a = 0.5  # Reduce alpha for glow effect
```

## Pending Conversions

### **Next Steps for Wizard Conversion**
1. ❌ **Boss Enemy**: Convert ship6 → wizard sprite (4th wizard type needed)
2. ❌ **Exhaust Effects**: Replace ship exhaust → wizard spell effects  
3. ❌ **Shot Sprites**: Replace laser shots → magical projectiles
4. ❌ **Animation Frames**: Implement wizard walking/casting animations

### **Future Animation Enhancements**
- **Wizard Walking**: Use walk_1.png, walk_2.png, walk_3.png, walk_4.png
- **Wizard Casting**: Use attack_1.png, attack_2.png, attack_3.png  
- **Wizard Idle**: Use idle_1.png, idle_2.png, idle_3.png, idle_4.png
- **Wizard Death**: Use dead_1.png, dead_2.png, dead_3.png, dead_4.png

## Technical Notes

### **Godot Resource Loading**
- **ext_resource**: Defines sprite file paths in .tscn files
- **uid**: Unique identifier for resource tracking  
- **region_rect**: Defines sprite regions for texture atlases
- **modulate**: Runtime color/alpha modification

### **Node Tree Structure**
```
Player (CharacterBody2D)
├── Ship (Sprite2D) ← Main wizard visual
├── ShipGlow (Sprite2D) ← Energy glow effect
├── Exhaust (AnimatedSprite2D) ← Movement effects  
├── ShipExplosion (AnimatedSprite2D) ← Death animation
└── ShotOutEffect (AnimatedSprite2D) ← Shot effects
```

### **Performance Considerations**
- **Sprite Sheets**: Individual files vs. texture atlases
- **Animation Caching**: SpriteFrames resources for repeated animations
- **Memory Usage**: Multiple texture references in scene files

---

*Report Generated: Wizard Conversion Phase 1 Complete*  
*Status: Ship sprites converted, projectiles and effects pending* 
