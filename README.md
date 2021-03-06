# Convert Number Properties into a Dynamic Moon Emojis Rating Scale with Decimal Support

There are lots of tutorials about how to make your own dynamic emoji ratings, but I have found none that takes into consideration values with decimals like 3.5, 2.2, 4.75, etc. Thus, I came up with one myself.

This emoji ratings formula is the first Notion formula I wrote, and although I admit itโs quite convoluted, it works anyway, so who cares. If you know how to do better, then feel free to modify it.

We will be using a **rating scale of 1-5**, and values can have up to **two decimal places**. If it goes beyond two decimal places, the emojis won't show up properly. (The formula can be adjusted to accommodate more decimal places, but since this is only for book ratings, I will forego the hassle). And lastly, we will be using the different moon phases emojis: ๐๐๐๐๐

---

## Tutorial

### Preparing the Database

**Step 1.** In your database, create a number property and name it โNumber Rating.โ This number property is where you will input the numerical ratings. You may choose to hide this in your database view later if you want to show only the final moon ratings. [For this tutorial, I suggest you use the table view to make it easy to follow along.]

**Step 2.** Next, create a formula property. The name here doesnโt matter. In my setup, I call it Moon Ratings.


**Step 3.** Paste the entire **emojis-only.txt** formula into the formula property if you want to show only the emojis. Use **numbers-and-emojis.txt** if you want the rating value to show up beside the emojis.

**Step 4.** Enter some numbers in the number property column and watch the emojis show up. And youโre done!

If the number property is empty, then it will show an โN/A.โ

---

## How it Works

The formula is divided into 7 stages.

**Stage 1.** This part defines what happens if the number property has no value:

    empty(prop("Number Rating")) ? "N/A" 

You can change the โN/Aโ here to your own custom text.

**Stage 2**. This part determines how many full moons ๐ to return:

    (slice("๐๐๐๐๐", 0, floor(prop("Number Rating")) * 2)

**Stage 3.** This stage intersects with stage 4. A lot of stuff happens in this part.

First, it checks if the value contains a decimal by extracting the decimal numbers using `slice`. Second, it checks if it has 1 or 2 decimal places (which is important for the formula to be able to handle values having 2 decimal places). And lastly, it excludes values having no decimals from being implemented in Stage 4.

    format(smaller(equal(length(slice(format(prop("Number Rating")), 2)), 1) ?

**Stage 4.** Since stage 4 intersects with stage 3, you can see stage 3 still included below. Stage 4, particularly starting from the second `toNumber` statement, defines the conditions for a waning crescent moon emoji ๐ to be returned.

    format(smaller(equal(length(slice(format(prop("Number Rating")), 2)), 1) ? (toNumber(slice(format(prop("Number Rating")), 2)) * 10) : toNumber(slice(format(prop("Number Rating")), 2)), 40) ? "๐" : "")

The number 40 here tells us that values with a decimal of less than 0.4 should get the ๐ emoji. If the value has no such decimal, then nothing will happen. The formula will then continue to the next stage.

You can adjust the 40 here to a bigger or a smaller number if you like, but by doing so, you will also have to adjust what number range would give a ๐ or ๐ emoji

**Stages 5 and 6** defines the conditions for when a half-moon ๐ or a waning gibbous moon ๐ emoji, respectively, is returned.  Each of these two stages is a combination of stages 3 and 4. The only thing that makes them different (and slightly more complicated) is that you have to define a **lower and upper limit**.

**For Stage 5.** You have to state that a half-moon only appears when the decimal number is **larger or equal to 40** (0.4) **but less than 70** (0.7). You can customize the lower and upper limit here, but make sure to adjust the limits in other places in the formula as well.

    format((largerEq(equal(length(slice(format(prop("Number Rating")), 2)), 1) ? (toNumber(slice(format(prop("Number Rating")), 2)) * 10) : toNumber(slice(format(prop("Number Rating")), 2)), 40) and smaller(equal(length(slice(format(prop("Number Rating")), 2)), 1) ? (toNumber(slice(format(prop("Number Rating")), 2)) * 10) : toNumber(slice(format(prop("Number Rating")), 2)), 69)) ? "๐" : "")

**For Stage 6**. A waning gibbous moon emoji ๐ is returned only when the decimal place is **larger or equal to 70** (0.7) and when it is **smaller or equal to 99**.

    format((largerEq(equal(length(slice(format(prop("Number Rating")), 2)), 1) ? (toNumber(slice(format(prop("Number Rating")), 2)) * 10) : toNumber(slice(format(prop("Number Rating")), 2)), 70) and smallerEq(equal(length(slice(format(prop("Number Rating")), 2)), 1) ? (toNumber(slice(format(prop("Number Rating")), 2)) * 10) : toNumber(slice(format(prop("Number Rating")), 2)), 99)) ? "๐" : "")

**Stage 7.** Lastly, this part tells us how many new moon emojis ๐ to return

    slice("๐๐๐๐๐", 2 * ceil(prop("Number Rating"))))


## Snippets

**Display Zeroes in Decimals**

    equal(length(slice(format(prop("Number Rating")), 2)), 2) ? format(prop("Number Rating")) : equal(length(slice(format(prop("Number Rating")), 2)), 1) ? concat(format(prop("Number Rating")), format(0)) : equal(length(slice(format(prop("Number Rating")), 2)), 0) ? concat(format(prop("Number Rating")), ".", format(0), format(0)) : ""

---

**Airtable Version**

    IF(Rating=BLANK(),"N/Aโ๐๐๐๐๐",CONCATENATE(Rating,"โ",REPT("๐",FLOOR(Rating)),IF(AND((RIGHT(Rating,2)>0),(RIGHT(Rating,2)<=40)),"๐",""),IF(AND((RIGHT(Rating,2)>40),(RIGHT(Rating,2)<70)),"๐",""),IF(AND((RIGHT(Rating,2)>=70),(RIGHT(Rating,2)<=99)),"๐",""),IF(RIGHT(Rating,2)>0,REPT("๐",FLOOR(5-Rating)),REPT("๐",5-Rating))))
