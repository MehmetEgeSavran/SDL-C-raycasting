TO COMPILE THE CODE,
mingw32-make

TO RUN THE CODE,
./main


|================| SDL 2 Tutorial |================|

|================| 1. BASICS |================|
------------------------------------------
-- always start with include files,
-- next set your screen dimension constants,
    const int SCREEN_WIDTH = 600;
    const int SCREEN_HEIGHT = 480;

-- the entry point is int main,
-- however it has the parameters int argc, char* args[],
    int main( int argc, char* args[]) {...}

1.1 Arguments
------------------------------------------
-- when a program run from the Command Line, Terminal or Console, 
   you provide additional information, or arguments that the program uses,

-- for example,
   ./main.c arg1 arg2 arg3
   ./main.c : is the name of the program/executable files
   arg1, arg2, arg3 : are the arguments we are passing to the program

-- Arguments representes as an array, the first element of this array,
   args[0] is always the name (or path) of the program itself,
   args[0] = "./my_program"

-- here is a simple code to illustrate the arguments,
    #include <stdio.h>

    int main(int argc, char* args[]) {
        // Print the number of arguments
        printf("Number of arguments: %d\n", argc);

        // Print each argument
        for (int i = 0; i < argc; i++) {
        printf("Argument %d: %s\n", i, args[i]);
        }

        return 0;
    }

-- if you run this command as,
   ./my_program hello world

-- the output would be,
   Number of arguments: 3
   Argument 0: ./my_program
   Argument 1: hello
   Argument 2: world

1.2 Structure of a Basic SDL code
------------------------------------------
-- the basic structure looks like this,
    int main( int argc, char* args[]) {
        SDL_Window* window = NULL;

        SDL_Surface* screenSurface = NULL;

        if( SDL_Init(SDL_INIT_VIDEO) < 0) {
            pritf("SDL Could Not Initialize, SDL_ERROR: %s \n", SDL_GetError());
        }
    }

-- The contents are,
   SDL_Window : The window we are rendering onto
   SDL_Surface : The surface contained by the window
   if(SDL_Init(SDL_INIT_VIDEO) < 0) {...} : basic initialzation with debugging output

-- SDL_Surface is just a 2D image, it can be loaded from a file or 
   it can be the image inside of a window,

-- We cannot call any SDL functions without initializing SDL,
-- If there is an ERROR, the SDL_Init returns -1,

-- Also SDL_GetError() functions is used quite often, 
   whenever we want to debug, we call the function
-- for example,
    ...
    else {
        //Window creation
        window = SDL_CreateWindow (
            "SDL_Tutorial",
            SDL_WINDOWPOS_UNDEFINED, 
            SDL_WINDOWPOS_UNDEFINED, 
            SCREEN_WIDTH,
            SCREEN_HEIGHT,
            SDL_WINDOW_SHOW
        );
        if(window == NULL) {
            printf("An error has been encountered during window creation, SDL_ERROR: %s\n" SDL_GetError());
        }
    }


1.2.1 Pointer Creation
------------------------------------------
-- as you can already notice, most parameters like windows or screen surfaces
   are created as pointers. There are 4 main reasons,

   a. Dynamic Memory Management:
      when a window or surface is created, we don't know how much memory is needed, the 
      memory may need adjustments before and during runtime, thus Dynamic Memory Management
      is needed
   b. Performance:
      windows and surfaces can be large structures, passing them with pointers are more 
      efficient since passing them with pointers only requires their address instead of the 
      entire structure
   c. Null Pointer Check:
      from an error handling perspective, to check if a window or a surface creation is 
      failed, we deduce the result by checking with a NULL pointer, 
            if(window == NULL) {...}
   d. Manipulation of Shared Resources:
      Using pointers allows multiple parts of the program to reference and manipulate 
      the same window or surface. If we just coppied the object, it would instead created
      multiple instances and decreased the performance considerably 

1.3 Window Creation
------------------------------------------
-- As seen above the window creation function is,

   SDL_Window* window = SDL_CreateWindow(
    "Window Name",
    SDL_WINDOWPOS_UNDEFINED,
    SDL_WINDOWPOS_UNDEFINED,
    SCREEN_WIDTH, SCREEN_HEIGHT,
    SDL_WINDOW_SHOW
   );

-- the 2nd and 3rd arguments of this function are for when the x and y position of the
   window is irrevelant,
-- last argument, SDL_WINDOW_SHOW makes sure that the window is shown to the user when
   it is created,
-- if there is an error during creation, it is printed via the SDL_GetError() and printf
   functions,

   if(window == NULL) {
    printf("An Error Has Been Encountered During Window Creation, SDL_ERROR: %s\n", SDL_GetError());
   } else {
    ...
   }

-- if no errors are encountered during creation, the program can move towards the else 
   condition,

   else {
    screenSurface = SDL_GetWindowSurface(window);
    SDL_FillRect(
        screenSurface,
        NULL,
        SDL_MAPRGB(
            screenSurface -> format, 0xFF, 0xFF, 0xFF
        )
    );
    SDL_UpdateWindowSurface(window);
    SDL_Event e; 
    bool quit = false;
    while(quit == false) {
        while(SDL_PollEvent(&e)) {
            if(e.type == SDL_QUIT) 
                quit = true;
        }
    }
   }

-- the explaination of the code above is,
   screenSurface = SDL_GetWindowSurface(window) : we combine the surface to the window
   SDL_FillRect(...) : to fill the surface
   SDL_UpdateWindowSurface : an important part of this code piece is that, even though we 
        have drawn someting on the surface, we cannot see the new materials if we do not
        update the window surface
   
-- to keep the window closing after updating it, we need to keep it from disappearing. To 
   do this, we could use SDL_Delay, or as seen above a piece of code that keeps the code 
   open

-- after using the window we move on to deallocating the memory, and finishing update
   
    // To Destroy a Window
    SDL_DestroyWindow(window);
    //Quitting SDL Subsystems
    SDL_Quit();
    return 0;

|================| 2. IMAGE RENDERING |================|

-- dividing the main entry point function, or main function into smaller functions is called
   making the code more "modular",
-- that is why on this tutorial we take a look at this simple modularity functions,
    bool init();        : starts SDL and creates a window
    bool loadMedia();   : loads media
    void close();       : frees the loaded media and shuts down SDL

-- the setup is similar to the examples before,
    SDL_Window* window = NULL;          : just as before
    SDL_Surface* surface = NULL;        : just as before
    SDL_Surface* imageToLoad = NULL;    : is the parameter that 
                                          references to the to be loaded image 
-- also on an important note, the SDL_Surface's surfaces use software rendering,
   which uses the CPU to render the image. GPU accelerated image rendering will be handeled later
-- on another note, always Initialize pointers,

-- the initialzation function is,
    bool init() {
        bool success = true;
        if( SDL_Init(SDL_INIT_VIDEO) < 0) {
            printf("SDL Could not Initialize becasue of an error, SDL_Error: %s\n", SDL_GetError());
            success = false;
        } else {
            window = SDL_CreateWindow(
                "SDL Window",
                SDL_WINDOWPOS_UNDEFINED,
                SDL_WINDOWPOS_UNDEFINED
                SCREEN_WIDTH, SCREEN_HEIGHT,
                SDL_WINDOW_SHOW
            );
            if(window == NULL) {
                printf("An error has been encountered during window creation, SDL_Error: %s\n", SDL_GetError());
                success = false;
            } else {
                screenSurface = SDL_GetWindowSurface(window);
            }
        }
        return success;
    }

-- the image loading function is,
    bool loadMedia() {
        bool success = true;
        imageToLoad = SDL_LoadBMP("image.bmp");
        if(imageToLoad == NULL) {
            printf("Failed to load the image due to an error, SDL_Error: %s\n", SDL_GetError());
            success = false;
        }
        return success;
    }

-- the deallocation and enclosure function is,
    void close() {
        SDL_FreeSurface(imageToLoad);
        imageToLoad = NULL;
        SDL_DestroyWindow(window);
        window = NULL;
        SDL_Quit();
    }

-- and the final structure of the main function would resemble this,
    int main(int argc, char* args[]) {
        SDL_Window* window = NULL;          
        SDL_Surface* surface = NULL;        
        SDL_Surface* imageToLoad = NULL;    
        if(!init()) {
            printf("Failed to Initialize!\n");
        } else {
            if(!loadMedia()) {
                printf("Failed to load media!\n");
            } else {
                SDL_BlitSurface(imageToLoad, NULL, screenSurface, NULL);
                SDL_UpdateWindowSurface(window);
            }
        }
    }

|================| 3. EVENT DRIVING |================|

-- besides rendering, SDL also handles user inputs,
    ...
    bool quit = false;
    SDL_Event = e;
    ...
-- the quit flag tracks whether the user demanded quitting or not,
-- an SDL_Event could be a key press, mouse motion etc etc,
    ...
    while (!quit) {
        if(e.type == SDL_QUIT) {
            quit = true;
        }
    }
    ...

-- when the user gives an input, aka an event, the program puts the events in an 
   "EVENT QUEUE". The queue will store them in order of occurence. 
-- To get the most recent event from the event queue, we poll the queue with SDL_PollEvent,
   SDL_PollEvent will keep taking events from the queue until the queue is empty, then it 
   returns 0,

|================| 4. KEY PRESSES |================|

-- enumeration of keyboard inputs is,
    ...
    enum KeyPress {
        KEY_PRESS_SURFACE_DEFAULT;
        KEY_PRESS_SURFACE_UP,
        KEY_PRESS_SURFACE_DOWN,
        KEY_PRESS_SURFACE_LEFT,
        KEY_PRESS_SURFACE_RIGHT,
        KEY_PRESS_SURFACE_TOTAL
    };
    ...
-- the integer equivalent of the keyboard constants are, 
    KEY_PRESS_SURFACE_DEFAULT = 0
    KEY_PRESS_SURFACE_UP = 1
    KEY_PRESS_SURFACE_DOWN = 2
    KEY_PRESS_SURFACE_LEFT = 3
    KEY_PRESS_SURFACE_RIGHT = 4
    KEY_PRESS_SURFACE_TOTAL = 5

-- an example piece of code would be, 
    ...
    bool init();
    bool loadMedia();
    ...
    void close();
    SDL_Surface loadSurface()
    //SDL_Surface* imageToLoad = loadSurface( //string path );
    SDL_Window* window = NULL;
    SDL_Surface* screenSurface = NULL;
    SDL_Surface* keyPressSurface [ KEY_PRESS_SURFACE_TOTAL ];
    SDL_Surface* currentSurface = NULL;
    ...
    SDL_Surface* loadSurface ( //string path) {
        SDL_Surface* loadedSurface = SDL_LoadBMP(path.c_str());
        if(loadedSurface == NULL) {
            printf("Unable to load image with path: %s, SDL ERROR: %s\n", path, SDL_GetError());
        }
        return loadedSurface;
    }
    ...

|================| 5. Texture Loading and Rendering |================|

-- texture rendering provides accelerated hardware rendering,
    ...
    bool success = true;
    SDL_Texture* loadTexture(//string path);
    SDL_Window* window = NULL;
    SDL_Renderer* renderer = NULL;
    SDL_Texture currentTexture = NULL;
    ...

-- when we are working with textures, we need a renderer to render the image onto a window
    ...
    window = SDL_CreateWindow(
        window,
        SDL_WINDOWPOS_UNDEFINED,
        SDL_WINDOWPOS_UNDEFINED,
        SCREEN_WIDTH, SCREEN_HEIGHT,
        SDL_WINDOW_SHOW
    );
    if(window == NULL) {
        printf("An Error has been encountered during window creation, SDL_Error: %s", SDL_GetError());
        success = false;
    } else {
        renderer = SDL_CreateRenderer(
            window,
            -1,
            SDL_RENDERER_ACCELERATED
        );
        if(render == NULL) {
            printf("An Error has been encountered during renderer creation, SDL_Error: %s", SDL_GetError());
            success = false;
        } else {
            SDL_SetRendererDrawColor(
                renderer, 
                0xFF,
                0xFF,
                0xFF
            );
            int imageFlags = IMG_INIT_PNG;
            if( !(IMG_Init(imageFlags) & imageFlags) ) {
                printf("SDL_Image could not be loaded, SDL_Image_Error: %s", IMG_GetError());
                success = false;
            }
        }
    }


|================| PERSONAL NOTES |================|