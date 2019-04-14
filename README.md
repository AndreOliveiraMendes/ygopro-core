# YGOPro script engine.

## Introduction
The core logic and lua script processor of YGOPro. This library can be made external of the project and used to power server technologies. It maintains a state engine that is manipulated by Lua scripts using manipulation functions it exposes.

## Compiling
### 1.) Download Fluorohydride/ygopro
Start by downloading the most parent of the source code. The team developing this project are the defacto edge and experts in our community. The most upto date `ocgcore` is a compiled dll version of the `Fluorohydride/ygopro/ocgcore` folders project.

### 2.) Install Premake4 and Visual Studio 2010 (or later).
Download premake5.exe, put it in `c:\windows` or a similar folder that is globally accessible via `cmd` or PowerShell. On Windows you need Visual Studio.

### 3.) Download dependencies
Dependencies are absent from the main project. There is information on how to build them but the easiest thing to do is to download the following folders from a [soarqin/ygopro](https://github.com/soarqin/ygopro) fork and simply copy them into the `Fluorohydride/ygopro` folder.

* event
* freetype
* irrlicht
* lua
* sqlite3

### 4.) Create the project files
Run the following commands from the command line in the `ygopro` folder.

` premake5 gmake ` if you're not using Visual Studio

` premake5 vs* ` (with the current edition of Visual Studio installed on your system)
there are also premake4 scripts that are synced with teh premake5 ones to use on systems where premake5 isn't availabe, or just as preference, but note that with visual studio builds, the max edition supported by those scripts is 2010.

### 5.) Build the system
If you are on windows using Visual Studio, and you used premake4, first right click on ` ygopro ` in teh solution explorer, and then set it as startup project.
To build with Visual Studio click on build->build solution, or click local windows debugger that will first compile the program then launch it.
On other systems, go with the terminal in the ` build ` folder, then run ` make config=release ` to build the static program.

## Exposed Functions

These three function need to be provided to the core so it can get card and database information.
- `void set_script_reader(script_reader f);` : Function used by the api to load a lua script corresponding to a card, the function structure is
  ```
  unsigned char* (const char* filename, int* buffer_length)
  ```
  The first parameter is the name of the script to load, it's structured in this way: `c(id of the card that has to be loaded).lua`, the buffer_length value has to be set with the total size of the buffer returned by this function, to make the api load nothing, set this value to 0. The return value is a pointer to the buffer containing the lua script loaded into memory, make sure that this buffer is declared as static or stored somewhere to avoid it being corrupted.
  The api has a built in script reader that loads the scripts from the ` script ` folder in the working directory.

- `void set_card_reader(card_reader f);` : Function used by the api to get the informations about a card, the function structure is
  ```
  uint32 (uint32 code, card_data* data)
  ```
  The first parameter is the id card to load, it's structured in this way: `c(id of the card that has to be loaded).lua`, the ` data ` parameter is a pointer to a struct that has to be filled with the various stats of the card.
  
  ```cpp
  struct card_data {
    uint32 code;
    uint32 alias;
    uint64 setcode;
    uint32 type;
    uint32 level;
    uint32 attribute;
    uint32 race;
    int32 attack;
    int32 defense;
    uint32 lscale;
    uint32 rscale;
    uint32 link_marker;
  };
  ```
  The return value isn't used.

- `void set_message_handler(message_handler f);` : Function used by the api to send to the host any state message generated by teh api, the function structure is
  ```
  uint32 (void* duel, int type)
  ```
  The fist parameter is a pointer to the duel object that generated that call, that pointer has to be used then in the ` get_log_message ` function to get the actualcontent of the message. The second parameter specifies the nature of teh message, 1 means that it is due an error in the lua machine, 2 means it's currently a debug message generated via a script with the proper function.
  The return value isn't used.

These functions create the game itself and then manipulate it:
- `void* create_duel(uint32 seed);` : Create a duel instance using the provided value as seed for the internal random number generator.
  The return value is a pointer to the created duel.
- `void start_duel(void* pduel, int32 options);` : Starts the duel corresponding to the passed object, the second parameter are the various options to use in that duel, they can be found here https://github.com/edo9300/ygopro-core/blob/master/common.h#L377.
  Call this function everything has been loaded in the current instance, so the players' decks, the various info etc.
- `void end_duel(void* pduel);` : Ends the duel corresponding to the passed object, after that the object is destroyed and thus the pointer is invalidated.
- `void set_player_info(void* pduel, int32 playerid, int32 lp, int32 startcount, int32 drawcount);` sets the duel up
- `void get_log_message(void* pduel, byte* buf);`
- `int32 get_message(void* pduel, byte* buf);`
- `int32 process(void* pduel);` : do a game tick
- `void new_card(void* pduel, uint32 code, uint8 owner, uint8 playerid, uint8 location, uint8 sequence, uint8 position);` : add a card to the duel state.
- `void new_tag_card(void* pduel, uint32 code, uint8 owner, uint8 location);` : add a new card to the tag pool.
- `int32 query_card(void* pduel, uint8 playerid, uint8 location, uint8 sequence, int32 query_flag, byte* buf, int32 use_cache);` : find out about a card in a specific spot.
- `int32 query_field_count(void* pduel, uint8 playerid, uint8 location);` : Get the number of cards in a specific field/zone.
- `int32 query_field_card(void* pduel, uint8 playerid, uint8 location, int32 query_flag, byte* buf, int32 use_cache);`
- `int32 query_field_info(void* pduel, byte* buf);`
- `void set_responsei(void* pduel, int32 value);`
- `void set_responseb(void* pduel, byte* buf);`
- `int32 preload_script(void* pduel, char* script, int32 len, int32 scriptlen = 0, char* scriptbuff = nullptr);`

# Lua functions
`interpreter.cpp`
