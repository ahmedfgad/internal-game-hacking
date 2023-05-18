# Game Hacking

Skills needed:

1. Memory Editing (Mandatory): Keep the game code running but edit its memory.
2. Assembly Editing (Advanced): Rewrite the game's code to do what you want to do.
3. Hex Editing (Rarely used): Edit save-files or other files on the disk. Memory editing is better. But for console games, memory editing is not an option and we have to use hex editing.
4. Packet Editing: Used for online games. Edit the packets as they are sent to the server. 
5. Botting: Write a bot to play the game for you.

To explain the difference between memory and assembly editing, consider that you want to create God Mode. In memory editing, you edit your health every some milliseconds to increase the health to 100. If you took damage, then the health is restored by editing the memory. But sometimes it might happen that you took a damage greater than 100. Thus, you will die before editing the health in the memory.

For assembly editing, you rewrite the code that makes sure the health never drops below 100. This is by finding the code that hurts the player and disable it.

## Tools

The hacking tools include:

* Cheat Engine: Developed in 2003.
* [Process Monitor](https://learn.microsoft.com/en-us/sysinternals/downloads/procmon): Monitor the processes. Used for Hex Editing to locate the save files.
* [HxD](https://mh-nexus.de/en/hxd): Hex editor. Used for Hex Editing to search for strings and values.

Squally is a game that is developed by Guided Hacking for hackers to get started. https://store.steampowered.com/app/770200/Squally

These tools are using the operating system API to change the memory of the running processes. But you have to tell the OS what and where exactly to make the change. For example, the Windows API has a C++ function called [WriteProcessMemory](https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-writeprocessmemory) that writes the memory of a process. 

The process of hacking a game includes:

1. Scanning the memory: Find exactly the memory address to edit.
2. Editing the memory.

## Hex Editing

https://youtu.be/EpcK8uk7lcY

Hex editing is mainly used to edit save files. But the skills learned in Hex editing helps with resource, memory, and packet editing.

The steps are:

1. Find the info in the save file to edit.
2. Hex Edit the save file.

When a game saves files to the disk, then it makes requests to the OS. We can intercept these requests to see what files the game is creating by using a software like [Process Monitor](https://learn.microsoft.com/en-us/sysinternals/downloads/procmon). It shows the operations done by each running process. Examples of these processes include:

* ReadFile: When the process reads a file.
* CreateFile: When the process creates a file.

### String Search

Inside the game, make an action that causes a save file to be created for example. The entry in **Process Monitor** shows the full file path. Then go to this file. 

It is a good idea to have a copy of the save file in case you did something wrong (expected to make mistakes). Then edit the file to see if the changes are correct. Otherwise, restore the save file and make another try.

The save file is a hexadecimal file. So, it is not human-readable. Use a text editor like [HxD](https://mh-nexus.de/en/hxd) which converts the hexadecimal code into a string. From that string, we can find something helpful to find what we want. 

For example, if inside the game you changed the number of **golds** you have, then it is likely that the string will have the word **gold**. Use the hex editor [HxD](https://mh-nexus.de/en/hxd) for searching. 

It is also expected that the gold value is saved into a 4-bytes (INT32) integer that follow this word. Try changing some 4-bytes integers and see if this is successful. Watch [this video](https://youtu.be/EpcK8uk7lcY) for an example. This is an example of 4-bytes Hex number that is equivalent to 999 in decimal:

```
E7 03 00 00
```

But it is not guaranteed that the save file will have a label that guides us to the proper value. This depends on the file format defined by the game developer. For example, the file may store the data as a JSON file where each value has a descriptive key.

```json
{"items": {"num_gold": 50, "num_diamond": 20}}
```

In another case, the save file might not have labels at all. It can be saved in a sequence where each line has a value for an item. The first value is the number of gold and the second is for the number of diamond.

```
50
20
```

So, instead of using **string** search, it is better to use **value** search.

### Value Search

To search the save file by value, first we need to know the value from the game that we are looking for. For example, the decimal number of gold is 679.

Then convert this decimal number into hexadecimal to be **2A7**. This is the hexadecimal number if Big Endian byte order. When written as 4 bytes, then it is `02 A7 00 00`. If the file is saved in Little Endian order, then the number is `00 00 A7 02`.

Then search the file with this number. Once found, then then edit it. Use the hex editor [HxD](https://mh-nexus.de/en/hxd) for searching.

If there are so many matches of the target value, then follow the following strategy:

1. In the save file, search by the current value. Copy this file.
2. Inside the game, change the value (e.g. health). This causes the file to change according to the new value.
3. Compare the 2 files in the previous 2 steps. Find the locations where the file changed. If there are many matches, then repeat the 3 steps.
4. If there are no many matches, then it is likely that you found the target value. Edit the value and check the game to see if the target value changed successfully.

#### Limitations of Hex Editing

As explained earlier, Hex Editing depends on the save file format when searching by string. Even when searching by value, it may not work sometimes. This is because the game may have been developed in a way that changes the order of the values. At one time, it may save the enemy data before the player data. In another case, it may save the player data before the enemy data.

The files may get corrupted after being edited because the game may use anti-cheat like checksum to verify that the file is not changed. Moreover, the file itself may be encrypted.

## Memory Editing

To edit the memory of a game, we have first to do a **memory scan** to know the memory address to change. After that, we can change the value.

By memory scanning is different based on some factors like:

* Whether the value is visible. For example, the number of gold is displayed on the screen and you know exactly what the value is. In other cases, the value is not visible like the health bar. We do not know the exact health value. All we know is that the health increases and decreases.
* The data type. While scanning the memory, we have to select the data type of the target value. Then we can make changes in the value and monitor the changes. If a wrong data type is selected, then we are scanning the wrong memory addresses.

If the value is visible (e.g. number of gold), then we can simply enter the value in a memory scanner like Cheat Engine and scan for the value. Then change the value in the game and search for the new value in Cheat Engine. Do this more and more until you end up with just few values. For the optimal situation, we will end up with a single value.

If the value is not visible (e.g. health bar or player XY position), then we have to select the target value (e.g. float). Then set the **Scan Type** to **unknown initial value**. Then click **First Scan**.

Then change the value in the game to something bigger or smaller. For the next scan, select the Scan Type as either **Bigger than** or **Smaller than** based on whether the value increased or decreased. Then click **Next Scan**.

Continue increasing or decreasing the value and scanning until ending up with a short list of memory addresses. If the list does not get smaller, then the strategy is to:

* Freeze half of the addresses and try changing the value in the game. If the value does not change, then the target memory address is within the frozen half. If not, then it is in the other half.

Once the target memory address is found, change it.

### Static vs Dynamic Memory

The addresses stored in the memory can be either static or dynamic. Static memory addresses are the same after restarting the game. Dynamic addresses change after restarting the game.

### Virtual Memory

https://youtu.be/aPNcEckD1Qk

When an executable (.exe) file is loaded into memory, then it is allocated its own virtual memory. In 32-bit systems, each program is allocated 2GB of RAM. The first part of the memory is used for storing the program code. This part is static. The second part is for storing the data which is dynamic.

For example, if you defined a variable like `int x=5;`, then this variable is stored in the dynamic memory and its address changes each time the game is started.

If a variable is defined as static `static int x=5;`, then its address does not change as it will be stored in the first memory part. 

Each object created by the code later will have its memory block.

#### Dynamic Memory

https://youtu.be/_W0xdVO8-j4

In dynamic memory, the allocated memory for the objects change each time the game runs.

The entry point to access the dynamic objects is using the base address of the program. Let's say the base address is `0xA78BA210`. Then the static memory addresses and objects created by the program start after this address.

For example, the program may have the player X and Y as static addresses and it creates the Player object. Each of these addresses are 4 bytes long.

If the base address of the program is `0xA78BA210`, then player X address is `0xA78BA210 + 0` and player Y address is `0xA78BA210 + 4`. The numbers added to the address are the offsets.

After these 2 addresses, the Player object is created at offset 8 `0xA78BA210 + 8`. But we do not know exactly how much of the memory is allocated for this object. So, we cannot decide the offset for the next object or variable we need to created.

Let's say that the Player object creates a new object called **Items** to represent the items the player have. How can we access this Items object?

It is not possible to access it using the memory offset because, as we said, we cannot determine the amount of memory allocated for the Player object. Therefore, we cannot decide the memory location of the objects following the Player object.

But the Player object will have a **pointer** that points to the memory location of the Items object. This is how we can access the Items object. 

If the Pointer is saved into the 12th byte of the Player object, then the pointer to the Items object is accessible by `(0xA78BA210 + 8) + 12`.

So, the chain is:

1.  Get the program base address.
2. Get the address of the Player object.
3. Get the pointer of the Items object.
4. Access the Items object using its pointer.

Note that **pointers** are used so that objects can access each other.

Tools like Cheat Engine scan all the pointers of memory to find the path for each dynamic variable. It may happen that there are multiple paths to the same dynamic variable. But some paths might be more reliable than the others.

#### Find Pointers using Cheat Engine

Let's assume that we used Cheat Engine to scan for the player health and we found its address. Because this address will change the next time the game runs, we can benefit from the current address to find the pointers to this address.

This is done by right clicking the address and click **Pointer scan for this address**. Change whatever you want or keep things to the default. Then click **OK** and select where to save the result of scanning.

The result will have so many paths to the pointer. Because not all of these paths are reliable, follow this strategy to filter the paths:

1. Restart the game. Keep the pointer scan window open.
2. Reattach the game process to Cheat Engine.
3. Find the target memory address again by scanning (e.g. the player health or XY position).
4. Once the address is found, then go to the opened pointer scan window.
5. From the **Pointer scanner** item in the menu bar, click **Rescan memory**.
6. Enter the memory address. We may also enter the offset of the pointer accessing the address by checking the **Must end with offsets** box to paste the offset.
7. click **OK** and select where to save the result of scanning.

Once scanning done and we have a short list of pointers, double click all the pointers to have them in the main Cheat Engine window.

If the last digit of the target memory address is not 0, 4, 8, or C, then we have to do the following:

1. In the memory scanner window, check the **Show advanced options** box.
2. Uncheck the **Addresses must be 32-bit aligned**.

This might be needed if the data type of the address is not a 4-bytes or float. For example, 2-bytes.

#### Cheat Engine Pointer Map

https://youtu.be/nQ2F2iW80Fk

To scan the memory for the pointers, the traditional way is to do a **pointer scan for this address**. To filter the results, you close the game and repeat the process again and again until having some few results.

Every time a pointer scan is done, then the game memory is scanned for all the pointers. The problem is that this scans every single address instead of scanning things that are pointers. This saves time.

To make things even faster, we can follow this strategy that uses a pointer map and the offset of the address accessing the target address:

1. Find the target memory address. Keep this address along the process.
2. Find the offset of the variable accessing that memory address. This is by right-clicking the address and clicking **Find out what accesses this address**. Make sure that the VEH Debugger is used. This is by going to `Edit > Settings > Debugger Options` and select **Use VEH Debugger**.
   1. Copy the offset of the variable that accesses the address. This will be used when making a pointer scan to narrow down the search space.
3. Right-click the address and click **Generate pointermap**. This pointermap will be used later when scanning for the pointers.
4. Right-click the address and click **Pointer scan for this address**.
   1. Check the **Use saved pointermap** box and select the saved pointer map. 
   2. Check the **Pointers must end with specific offset** box and paste the offset.
   3. Click **OK** and save the pointer file.
5. Before restarting the game, do the following to make it easier to find the new target address:
   1. Right-click the target address and click **Find out what accesses this address**. Make sure that the VEH Debugger is used.
   2. This opens a new window that is empty. Keep it open.
   3. Make the game window active (in front of other windows) and try to make any action that changes the value of the target address. Then pause the game.
   4. Switch to the opened window in CE and some stuff will be there. 
   5. Select a code line and click the **Add to the codelist** button. You can give the code a name and click **OK**. Then close this window.
   6. The code line can be accessed by clicking the **Advanced Options** from the main CH window. This is retained as long as we did not close CE. This code line will be used to get the new target address after restarting the game.
6. Close the game and open it again.
7. To get the target address after restarting the game, do the following:
   1. Click **Advanced Options** from the CE main window.
   2. Right-click any code line and select **Find out what addresses this code writes to**. Click **Yes**.
   3. A new window appears with the new address. You may have to get the game window active again before seeing something in this window.
   4. Double click the address to be added to the address list in the main CE window.
   5. If multiple addresses are returned in the window, then if you know the exact value (e.g. number of gold) of the target address, then sort the addresses by value and select the target address. But if the value is unknown (e.g. player X position), then we have to find the address using the traditional way.
8. Right-click the address and click **Generate pointermap**. Now we have 2 pointer maps.
9. Right-click the address and click **Pointer scan for this address**.
   1. Check the **Use saved pointermap** box and select the second saved pointer map. 
   2. Check the **Compare results with other saved pointermap(s)** box.
      1. Select the first saved pointer map.
      2. Select the old target address found in step 1. 
   3. Check the **Pointers must end with specific offset** box and paste the offset.
   4. Click **OK** and save the pointer file.
10. When the second pointer scan is done, then it is likely that the pointer list is shorter than the strategy of scanning with a single pointer map.
11. To further cut down the list, do the following:
    1. From the **Pointer scanner** item in the menu bar, click **Rescan memory**.
    2. Enter the memory address. We may also enter the offset of the pointer accessing the address by checking the **Must end with offsets** box to paste the offset.

To save the address list before close CE, click File then Save. Later, click Open to load the cheat table.

## Assembly Editing

https://youtu.be/_Sm84vARhbw



## Create Hot Keys in Cheat Engine

https://youtu.be/XPcpUweh5Es

## Create Scripts in Cheat Engine

1. https://youtu.be/BofjhNsR-FU
2. https://youtu.be/I9xO2mtpYgI

When a player gets damaged, then an instruction is executed that updates the memory address of the health.

To have the God mode where the health never decreases, there are 2 options:

1. Lock the health address value to a certain value (e.g. 100). But this will not save you from death because you may get damaged with more than 100. Thus, you will die before restoring the health value.
2. The better option is to disable the instruction that modifies the address. This is by:
   1. Finding which addresses access the address.
   2. Find the instruction that update the address. 
   3. Select the instruction and click **Show disassembler**.
   4. Click **Tools** then **Auto Assemble**.
   5. Click **Template** and then **Cheat Table framework code**.
   6. Click **Code injection**.
   7. Delete the instruction that updates the address from the script.
   8. Click **File** and then **Assign to current cheat table**. Without doing that, we will lose the script and have to repeat the process. 
   9. Now, the script is available in the address list. Double click it to open the script if you want.
   10. Lock the script to enable it.

Watch [the video](https://youtu.be/BofjhNsR-FU) for more details. Watch [this video](https://youtu.be/I9xO2mtpYgI) too.