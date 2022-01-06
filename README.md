# Bad Apple in Krunkscript

Step 1: Load the map here: https://krunker.io/?host=Bad_Apple This will load a cookie with the correct ID.

Step 2: Open devtools, navigate to application, then localstorage, then krunker.io. Search for "69" and observe the key's name. In my case it was "krkcst_140784_test", so we need to create a key called "krkcst_140784_rawdata". Make sure it's identical apart from rawdata.

Step 3: Download and open the provided display.txt file and copy the contents into the newly created rawdata key.

Step 4: Reload the map, and enjoy.

- Sometimes the display scale breaks, just reload the map if this happens.
- You need to click to start the video playback
- The ambient audio has timing cues to sync it up, make sure to click it at the correct time.

# Where's the source code???

If you're interested, here it is: https://github.com/LukeTheDuke240/badappledata/blob/main/client.krnk

# How did you do it??? + summarized notes

Since people like to put bad apple on anything and everything, from microwaves to the atari 2600, I thought "why not make it in krunkscript". Since krunkscript and javascript are similar I prototyped in JS, which was easily transferrable into krunkscript. I processed the video into many images using premiere pro, and made a small python script to batch process the images.

Tech wise, I got inspired by a geometry dash youtuber who created a level called "how", where he converted static images into shape arrays using a tool called geometrize. It's pretty cool ngl: https://www.geometrize.co.uk/, and it even comes with a command line version for easy scripting: https://github.com/Tw1ddle/geometrize-lib-example

Due to lack of hindsight on my part I was limited to 25 shapes per frame, at 15fps. It doesn't look good but it's recognizable at least. My ideal vision was embedding it in the script file but even this wasn't enough, since the maximum amount of data in a krunkscript file is around 350KB or so, and the final one is 1.5MB. Oh well. 











# Full notes (might be hard to read)

```
I thought of this project a few weeks ago but didn't decide to do it until now. Obviously putting bad apple on something has become something of a meme, and we've seen everything from atari 2600 to vector display ports.

Krunkscript supports modern media files such as png and mp3, but that's boring. I want the image to be dynamically generated and have its own flair. Obviously audio-wise we'll have to use an mp3 asset but this is a compromise I have to take due to no dynamically generated audio capability. There is an inherent size limit in krunkscript due to (what I assume are) memory limitations, and the script validator dies after like 15 seconds, so we cannot have anything over around 200KB.

One option is low-resolution pixellated image. Rendering as an array of pixels would allow me to use run-length encoding, greatly reducing the file size (and allowing a higher framerate/resolution) and removing the complicated JSON parsing.  I would only need to encode a 1bit value per pixel (maybe even optimizing to store 8 pixels in 1 byte) and this would be a much faster and less complicated workflow than shapes.

Rendering via shapes on the other hand have a much higher data requirement [type,xpos,ypos,xsize,ysize, color] OR [type,xpos,ypos,radius,color] so I'll likely need to compromise on framerate or number of shapes. Reducing the framerate to 12 or 15 would dramatically reduce the data requirement (and potentially allow more shapes) but I want to prioritize framerate here. If I wanted to go all out I could implement zip in krunkscript or some other compression algorithm since json is highly compressible, but not via run length encoding.

Both of these approaches have their pros and cons but since krunkscript supports js canvas shape rendering i want to take advantage of this.

Syncing the music with the video, and frame limiting will be a concern as well, maybe classic requestAnimationFrame trick with time.now

Bad Apple Video > ffmpeg to image sequence > imagemagick to 1bit png > geometrizer > optimize json (make 1bit as well) > parse JSON in krunkscript (options for serving it include arrays, decoding an image on krunker's servers, etc) 
Bad Apple Audio > upload asset to krunker's servers

(hard code any static values such as opacity or rotation if applicable, or optimize them to use as little data as possible (could do 0 for black rect, 1 for white rect, 2 for black circle, 3 for white rect. OPTIMIZE AS MUCH AS POSSIBLE - 30FPS IS THE GOAL

A good reference is "how i made how"






Geometrizer w/ rectangles and circles results in the best look. Theoretically i could crank up the number of shapes from 50 to like 200 for a better look. but I'm trying to prioritize frame rate here.

I could look at implementing a simple compression algorithm, such as run length encoding (which probably wont apply to shapes but to pixels) huffman encoding, lz77 or LZW. A dictionary encoder here (like lz77) would be ideal.  I might need to deal with raw hexadecimal for further optimization.

EDIT: Once it is optimized into an array, it has high entropy and cannot be compressed much. Compression should only be used with pixel type, run length encoding. Best way to compress json is serializing it to an array.








50 shapes, with rotated rects and ellipses on looks acceptable but could increase data requirement. I'll definitely need to scale down the number of shapes or the framerate.
1. I might be able to remove the first shape as it's the background of the canvas (might keep it static)
1.5: Will have to pre-process image into 1bit pngs so that geometrizer keeps color consistently white and black
2. Combine type and color (0 = black rect, 1= white rect, etc)








75 shapes looks perfect, with the following enabled: circles, ellipses, rectangles, rotated circles, rotated ellipses

0: black circle
1: white circle
2: black ellipse
3: white ellipse
4: black rectangle
5: white rectangle
6: black rotated rectangle
7: white rotated rectangle
8: black rotated ellipse
9: white rotated ellipse

background is always a rectangle covering the canvas of color 8c8c8c so it can be removed from the data and added to code
score can be omitted (i assume it's the amount that shape contributes to the similarity score)
opacity can also be removed as it's always 128

Geometrize picks the colors from the canvas so if it's pure black and white, you get pure black and white in return
I can preprocess the images through photoshop so i dont have to use image magick: boost contrast to 100, adjust brightness per scene for best detail










rectangle: type 1 [x, y, w, h]
rotated rectangle: type 2 [x, y, w, h, rot]
ellipse: type 8 [x, y, rx, ry]
rotated ellipse: type 16 [x, y, rx, ry, rot]
circle: type 32 [x, y, r]

rectangle
rotated r
ellipse
rotated ellipse
circle


type,color,params
color 0=black
1=white



Audio will be difficult.
Raw audio extracted from bad apple mp4 is 3.4MB
The max audio asset size is 200KB. My options are either keeping full quality and streaming it in 1 piece at a time (3.4/0.2 = 17 pieces) or compressing the audio small enough to.
Lowest quality VBR export from audacity (32kbps) is still 2MB and sounds like ass in an echo chamber.
Lowest possible export from audacity (8kbps) still isnt enough to fit the size limit (224kb) and it doesn't even resemble a song anymore, looks like we'll have to split the audio.
One potential option was using some trickery using mod files, and a trigger to change the ambient sound file but server side triggers are disabled. We have to split the audio file

64kbps is usable but it has artifacts. I wouldnt want to use this unless absolutely necessary.

MP3 has some issues including being virtually impossible to seamlessly splice together and encoder delay - an unavoidable 529 sample delay present on every MP3 decoder. This is worsened by using VBR
Since MP3 audio data is spread over several frames it's difficult to seamlessly splice without adding padding. We also need to use -nores on lame encoder to remove the bit reservoir so that seamless is possible.
Mod name is bad_apple_audio







Current workflow

Bad Apple Video -> premiere pro (contrast filter) -> export to png -> batch convert to json shape files using geometrizer -> compress using js or python -> add to krunkscript.
Bad Apple Audio -> Export using ffmpeg -> ambient sound mod

i will need to write some python code to parse each image on the command line and compress it, and some JS to parse it back into usable images (convert that code into simplified krunkscript)





I will probably have to re-export it from premiere to get rid of the starry background (might mess up geometrizer) 
Since we're going for maximum quality and frame rate here.



My approach will be loading the mod file (we'll need to start the animation through user input, which also means we can discard the first 30 or so frames of the video)

UPDATED 
0- white rectangle
1 white rotated rectangle
2 white ellipse
3 white rotated ellipse
4 white circle
5 black rect
6 black rot rect
7 black ellipse
8 black rot ellipise
9 black circle

combining color and shape innto the same byte saved 200 whole kilobytes holy shit


--input path=raw_bandw/ --output path=raw_json_output/ -t 1 -s 50 -a 128

-i input.png -o output.json -t "ellipse rectangle circle rotated_ellipse rotated_rectangle" -s 50 -c 250 -m 250

0123456789,[]

0000 0
0001 1
0010 2
0011 3
0100 4
0101 5
0110 6
0111 7
1000 8
1001 9
1010 ,
1011 [
1100 ]
1101
1110
1111

COMPRESSION
ascii printable characters 32 - 127
possibly extended ascii codes as well.

compression codes - pass 1
! = ],[      83k
@ = 0,       39k
# = 1,       30k
$ = 2,       35k
% = 3,       38k
^ = 4,       33k
& = 5,       32k
* = 6,       31k
( = 7,       35k
) = 8,       39k
- = 9,       34k

3 approaches
Key pair: guaranteed reduction by 2x === USE THIS APPROACH FIRST
Hex then key pair: unsure if it will fit in ascii
Number to symbol: potentially smallest





Storage.
1. Cookies (potentially the best)
2. Map storage
3. Embedded string DOESNT WORK

100KB WORKS
200KB WORKS
300KB WORKS
400KB TOO BIG


1. COokies (or more specifically the localstorage) works the best, can be modified at runtime, and krunkscript can load and parse it just fine. 

localStorage.setItem("krkcst_undefined_test", "String here")
```
