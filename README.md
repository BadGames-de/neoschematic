<div align="center">

### NeoSchematic

**Supports 1.18-1.21**

Original [Source](https://github.com/Efnilite/neoschematic) · [SpigotMC](https://www.spigotmc.org/resources/116652/) · by [Efnilite](https://github.com/Efnilite)

</div>

## Features
- Much easier and smaller than WorldEdit
- Schematics can be stored in JSON
  - Allows for easy editing and reading
  - Storage size scales linearly with the amount of blocks: 1 million blocks ~= 1 MB
- Schematics can be stored in ZIP
  - Allows for small file sizes

## Todo

- Add version migrations
- Add support for entities
- Add support for rotations

## Including in your project

### Maven
```xml
<repositories>
    <repository>
        <id>badgames-releases</id>
        <name>BadGames Repository</name>
        <url>https://repo.badgames.de/releases</url>
    </repository>
</repositories>

<dependencies>
    <dependency>
        <groupId>de.badgames</groupId>
        <artifactId>neoschematic</artifactId>
        <version>1.3.2</version>
        <scope>compile</scope>
    </dependency>
</dependencies>

<build>
<plugins>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-shade-plugin</artifactId>
        <version>3.3.0</version>
        <executions>
            <execution>
                <phase>package</phase>
                <goals>
                    <goal>shade</goal>
                </goals>
            </execution>
        </executions>
        <configuration>
            <relocations>
                <relocation>
                    <pattern>de.badgames.neoschematic</pattern>
                    <!-- Replace 'com.yourpackage' with the package of your plugin ! -->
                    <shadedPattern>com.yourpackage.neoschematic</shadedPattern>
                </relocation>
            </relocations>
        </configuration>
    </plugin>
</plugins>
</build>

```

### Gradle
```gradle
plugins {
    id 'com.github.johnrengelman.shadow' version '8.1.1'
}

repositories {
    mavenCentral()
    maven {
      name "badgamesReleases"
      url "https://repo.badgames.de/releases"
    }
}

dependencies {
    implementation 'de.badgames:neoschematic:1.3.2'
}

shadowJar {
    // Replace 'com.yourpackage' with the package of your plugin 
    relocate 'de.badgames.neoschematic', 'com.yourpackage.neoschematic'
}
```

### Manual
Copy the files into your project.

## Usage

### Create and save a schematic

```java
Schematic schematic = Schematic.create(pos1, pos2);
boolean success = schematic.save("plugins/schematic.json");
```

### Create and save a schematic asynchronously (Recommended)

```java
Schematic.createAsync(pos1, pos2, plugin).thenAccept(schematic -> 
    schematic.saveAsync("plugins/schematic.json", plugin)
);
```

### Paste existing schematic

```java
Schematic schematic = Schematic.load("plugins/schematic.json");
List<Block> blocks = schematic.paste(location, true);
```

### Load existing schematic asynchronously and paste (Recommended)

```java
Schematic.loadAsync("plugins/schematic.json", plugin).thenAccept(schematic -> {
    Bukkit.getScheduler().runTask(plugin, () -> schematic.paste(location, true));
});
```

### Example plugin

```java
package de.badgames.neoschematic;

import org.bukkit.Location;
import org.bukkit.block.Block;
import org.bukkit.command.Command;
import org.bukkit.command.CommandExecutor;
import org.bukkit.command.CommandSender;
import org.bukkit.entity.Player;
import org.bukkit.plugin.java.JavaPlugin;
import org.jetbrains.annotations.NotNull;

import java.util.Arrays;
import java.util.List;

public class ExamplePlugin extends JavaPlugin implements CommandExecutor {

    @Override
    public void onEnable() {
        getCommand("example").setExecutor(this);
    }

    @Override
    public boolean onCommand(@NotNull CommandSender sender, @NotNull Command command, 
                             @NotNull String label, String[] args) {
        if (args.length == 0 || !(sender instanceof Player player)) return true;

        switch (args[0]) {
            case "save" -> {
                var coords1 = Arrays.stream(args[1].split(",")).mapToInt(Integer::parseInt).toArray();
                var coords2 = Arrays.stream(args[2].split(",")).mapToInt(Integer::parseInt).toArray();

                var pos1 = new Location(player.getWorld(), coords1[0], coords1[1], coords1[2]);
                var pos2 = new Location(player.getWorld(), coords2[0], coords2[1], coords2[2]);

                sender.sendMessage("Getting blocks...");

                Schematic.createAsync(pos1, pos2, this).thenAccept(schematic -> {
                    sender.sendMessage("Saving file...");

                    schematic.saveAsync("plugins/schematic.json", this).thenRun(() -> {
                        sender.sendMessage("Schematic saved as schematic.json");
                    });
                });
            }

            case "paste" -> {
                sender.sendMessage("Loading file...");

                Schematic.loadAsync("plugins/schematic.json", this).thenAccept(schematic -> {
                    if (schematic == null) {
                        player.sendMessage("Schematic not found.");
                        return;
                    }

                    sender.sendMessage("Pasting blocks...");

                    getServer().getScheduler().runTask(this, () -> {
                        List<Block> blocks = schematic.paste(player.getLocation(), true);
                        player.sendMessage("Pasted %s blocks".formatted(blocks.size()));
                    });
                });
            }
        }

        return true;
    }
}
```
