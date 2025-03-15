# Process: Making astrophotography post-processing easier

## The rationale

Most of the information in a typical astrophotograph lies in a few narrow bands of brightess. I got frustrated trying S-curves and histograms and whatnot, and I wondered .... What if I could just pick the portions of the image that I wanted to highlight and work on them instead? 

For example, let's say I have a 16-bit image, but most of the pixel of interest have values between 4000 and 6000. Why can't I just say, "Map the range from 4000 to 6000 to a range of 30,000 to 50,000 in my new image"? If necessary, I can do this in multiple segments. And thus was born this project.

## Installation

You will need to install Julia <https://julialang.org/downloads/> and Jupyter Lab <https://jupyterlab.readthedocs.io/en/stable/getting_started/installation.html>. Once you've done that, import the notebook.

## Using the notebook

Open it. Then execute Cell 1. This will take a bit of time because it's loading several libraries. (On the todo list: Is it possible to import only parts of Images? It should be, but I haven't gotten that to work yet.)

Next, edit Cell 2 with the path name to your input image and execute the cell.

Then execute Cell 3. What Cell3 does is to create an array (named `meanne` because at one point it used means until I decided that wasn't a good idea) that's the size of the input picture; each entry in `meanne` is the maximum value of the red, green, and blue values of the corresponding pixel in the input image.

Now you'll want to find the useful regions of the image. The `blueify` function is your 
friend here. It takes three arguments: an input image, a lower bound, and an upper bound. There's an invocation of `blueify` in cell 4 that we'll be using for this.

The first thing you'll do with it is determine the black level of the image. To do this, use `blueify(img, 0, 1000)`. Increase or decrease the `1000` until the parts of the image that should be dark are all blue, but nothing else is. Make a note of the value you choose because you'll need it later on. I'll refer to this value as `LOW`.

The second thing to do is to find out what the maximum useful level of the image is. To do this, use `blueify(img, 30000, 65535)`. Increase or decrease the `30000` until the whole image is dark. The middle number should be as low as possible while keeping the whole image dark. (If you've got a few stars you don't mind overblowing, you can let them go blue.) Make a note of this number too. I'll refer to it as `HIGH`.

You'll probably want to divide the remaining range of the image into pieces. For example, if you're processing a picture of a galaxy, you might want to handle the core differently than you handle the arms. To do this, use `blueify(img, LOW, HIGH)`, replacing `LOW` and `HIGH` with the values determined above. Then you can either increase `LOW` until the arms are no longer blue or decrease `HIGH` until the core is no longer blue. Make a note of this value, which I will call `MID`.

You can split the image into as many ranges as you want, but I will stick with these three for now.

Move on down to Cell 5. Here, set `bottoms` to the low values for each of the ranges you've found so far: `bottoms = [0, LOW, MID, HIGH]`. 

The tricky part, and one that you'll have to experiment with, is the next set of numbers. What the program is going to do is to map input pixels the range from `0 to LOW` to whatever range you specify in `lowers`. You'll probably want the first range to be constant and fairly low, something like `1000`, so the first two numbers in `lowers` are both `1000`. For the next two numbers, I'd try something like `300000` (about half-full brightness) and `55000` (leaving a little headroom on the final image in case you're going to do something else to it).

This is going to:

* map all input pixels below `LOW` to the constant value of `1000`.
* map all input pixels between `LOW` and `MID` to the range from `1000` to `30000`.
* map all input pixels between `MID` and `HIGH` to the range from `30000` to `50000`.
* and map all pixels above `HIGH` to the range from `50000` to `65535`.

All the mappings are done linearly.

Now execute Cell 6. You probably won't like the first results. Change the values in  `lowers`. If a range is too dark, increase its lower end. If it's too compressed, give it more room. You can also change the input ranges in `bottoms` if you don't like your first choices, or add more ranges, etc. Re-run cell 5 every time you do this, and then Cell 6.

Now edit Cell 7 to contain the name of your output file and execute it. You're done!

