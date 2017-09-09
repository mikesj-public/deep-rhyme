# deep-rhyme

<img class="  wp-image-1620 aligncenter" src="https://swarbrickjones.files.wordpress.com/2016/11/1280px-parental_advisory_label-svg.png" alt="1280px-parental_advisory_label-svg" width="132" height="84" />

<strong>Disclaimer : I am a white boy, straight outta Cambridge, UK.  I do not condone the nastier, sometimes misogynistic language in this post, but in the interest of keeping it real, I have not made any effort to omit it either :b</strong>

<strong>Mobile users : you probably want to turn your screen sideways</strong>
<h2>Intro</h2>
The following was completely generated, unedited, by a rapbot (minus the title).
<pre><strong>House Full of Bricks - D-Prime</strong>

pump it up , just throw yo hands in the air
this goes out to all my niggaz that don't dare
i got a whole lot of diamonds in yo veins
about havin thangs gold rings , and my platinum chain
that's why all these bitch - ass niggaz get smoked
they don't give a fuck , that shit ain't no joke
nigga this goes out to all them crooked ass cops
since back in the days when i was getting popped
always had a good time for me to be fine
i've lost my mind , and i'm still gon shine
but i don't wanna see a nigga on the grind
we ain't trying to be ready for war or crime
check this out , a lot of people are scarred
and it's my time when it comes to the stars
cause i was born with a house full of bricks
yeah , we can see it all in the mix
but now it's hard for me to beg and feed
i gotta wake up , so take away my seed</pre>
Ever since I listened to <em><a href="https://en.wikipedia.org/wiki/Illmatic">Illmatic</a></em> as a youngster, I've loved hip hop, and the amazing creativity of (some) rap lyrics.  I also love machine learning.  Over a year ago, I decided I would write a machine learning algorithm that could ingest rap lyric text, then generate lyrics of its own automatically.  This was hard, and I gave up.  Twice.  After many, many iterations, I eventually came up with a model that could produce the above.  The following is a brief description of how this was achieved, the full gory technical details as well as code will be written up in a later post.
<pre>but right now i'm just trying to make it nice
this is my life , you can pay the price
i ain't gotta wait til it's time to take flight
have a party all night , everything's gonna be alright
so now do you really wanna get with me tonight
it ain't no need to talk about what i write</pre>
<img class="  wp-image-1686 aligncenter" src="https://swarbrickjones.files.wordpress.com/2016/11/cap3.jpg" alt="cap" width="311" height="316" />

<!--more-->
<h2>How this was done</h2>
<strong>The model</strong>

Generating free text using machine learning is not a new concept.  There was a huge surge in interest in it on the internet about a year ago because of this amazing <a href="http://karpathy.github.io/2015/05/21/rnn-effectiveness/">blog post by Andrej Karpathy</a>.  Text can be broken down into sequences, of characters, words, sentences etc.  There are some particularly powerful models called recurrent neural networks, that are designed to deal with general sequences.  The way these models are trained to learn about text goes like this - we give the network a short series of words (or characters) from real rap lyrics, such as
<pre>put your hands in the air , wave em like you just don't ____</pre>
and ask the model to predict what the next word in the sequence is.  Based on what the model thinks compared to the truth, we update the models' knowledge of rap language.  We repeat this for many millions of word sequences, and hope that eventually the model will learn a good understanding of the grammar and syntax used in the training data.

This approach conveniently allows us to generate arbitrarily long verses of free text - we give the model a starting sentence to work from, then generate the next word.  We assume this word is correct, then generate the next word, and the next.

This is a simplification - D-Prime uses a <a href="https://en.wikipedia.org/wiki/Beam_search">beam search approach</a> to generate lines - we generate a new line by building up many thousands of candidate lines word by word, keeping only the most promising ones at each step.  This is an idea lifted from machine translation, e.g. <a href="https://www.google.co.uk/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=0ahUKEwjkooPOyJbQAhWDbSYKHVH3DrIQFggiMAA&url=http%3A%2F%2Fwww.aclweb.org%2Fanthology%2FD13-1022&usg=AFQjCNFyH7WAn6GhHk7Ku6vkPWOoi7zrsw&sig2=IRkNDELFaS7WveTJdgPR6A">this paper</a>.  I also manually encoded some rules about what words could be used, to avoid repetitions etc.  I also forbid the model from choosing lines that were from the real rap lyrics, we want some originality here.

<strong>Data</strong>

Generally this kind of approach requires a lot of text to work effectively, i.e. a lot of lyrics.  These I obtained the lyrics of 50,000 rap songs off the internet using a python script.   I also spent a lot of time cleaning up the data, and trying to find the 'best' data to work from.  For example, I deliberately chose verses that had a rather mediocre vocabulary, meaning they mainly used commonly used words.  Incidentally, the song I found with the least mediocre vocabulary is <a href="http://genius.com/Cale-sampson-c-a-l-e-lyrics">this one</a>, but that's just showing off.

<strong>Rhyme and rhythm</strong>

I thought of the name DeepRhyme before I had even really started building the model. My brother pointed out 'D-Prime', and at that point I knew it was going to have to rhyme.

To achieve this I got the model to predict the text in reverse - that is we give it some reversed text, and ask the model to predict what word came before.  I can't speak for professional MCs, but if I need to make a rhyme, I do it like this - I have to think of the end of the line first.  Unfortunately, doing this alone was not enough to make the model rhyme on its own.  Instead, I forced it to.  Every other line, we start the next line off (at the end), by letting the model predict the probabilities for all possible next words, and only considering those that rhyme.

So how do we know what words go together?  In short- I downloaded the <a href="http://www.speech.cs.cmu.edu/cgi-bin/cmudict">CMU pronunciation dictionary</a>, and used a very basic set of hard coded rules to see if the end of words looked similar enough.  This was pretty loose, but rap lyrics are famous for using very slanted rhymes, listen to <a href="https://www.youtube.com/watch?v=ts-9p3O4TWM">Eminem</a> or <a href="https://www.youtube.com/watch?v=K-IRh7xSPrM">Mos Def</a> talk about what rhymes with orange for example.  I then looked at pairs of words found at the ends of lines in the actual rap lyrics themselves, and said that words rhymed if they appeared often as pairs in the lyrics, and passed my basic test.  This did not work perfectly - e.g. the model thought that og rhymed with dog, and not all words were even in the pronunciation dictionary - but it worked well enough.
<pre>and i'ma let you know that my game is thick
this rap shit , i do it for the bricks
cause none of y'all niggas can't fuck with my click
you know we ain't got no time for these tricks
so don't be mad cause the whole world is fake
it's a part of me , i'll do whatever i takes</pre>
Most of D-Primes rhyming is quite rudimentary, mainly just the <a href="https://en.wikipedia.org/wiki/Rhyme#Classification_by_position">tail rhymes</a> we have forced.  The best MCs use internal rhymes, as well as assonance and alliteration to make far more interesting syallble structure.  <a href="https://www.youtube.com/watch?v=ooOL4T-BAg0">This video</a> is a great illustration.  You can see that D-Prime does internally rhyme to a small extent, but making the model better in this regard seemed like it was going to be tough, and I didn't pursue it.

I also tried out an ABAB rhyme scheme, but it didn't really flow so well, I reckon rhymes have to be pretty on point for them to hold over two lines, and it's not really that common in hip hop anyway.  Here's one of the few sections that kind of worked.
<pre>ain't no need for us to do what we want
there's nothing in the world that can make me feel so sweet
i be out here tryin to get my hustle on
kept it real when they see me in the street
cause there's no way that i've been on my grind
but sometimes , i just wanna get out of control
feel like i'm the only one that's on my mind
and nobody wants to be a part of their soul</pre>
Rhythm, or 'flow', is an key differentiator of the best rap MCs.  Listen to the first verse of this :

https://www.youtube.com/watch?v=rFIrruzYWzU

I quickly decided I wasn't going to touch this concept with a barge pole.  Even working out how many syllables are in a line is pretty tough (e.g. many of the words are not in the pronunciation dictionary).  Instead I just told the algorithm to output lines that were exactly 10 words in length, including punctuation.  This was the average in the training set, and lead to the most pleasing text.

<strong>Early failed attempts</strong>

Because they're pretty funny, here's some bars written by earlier iterations of  D-Prime :
<pre>and i got a catch the streets, the streets are the streets
i can tell the streets, get the streets catch your shot, 
i can be but i can see the part
i was the streets, the streets in the streets
i spit a plant, the star and the motherfuckers, 
but i got a bottle of the streets in the back
the boots, i can take a control, i got the stars
i say the back of the streets, the fucking me, i was in the stars
i got a fly, i was the start, i got a back of the black start
i was the more than a discoll of the streets
i got a black, the streets is the streets stay back</pre>
Don't ever say D-Prime doesn't know where it came from.
<pre>cause i don't wanna be , no more , i'm like 
so i can go hard , i'm in the club 
i ain't got no time for a minute 
i can see my mama with her 
i can see what i can do 
i do my thing , i ain't been around 
but you didn't know , but now i know i know 
i ain't wanna , then i call him 
she said she was a bitch , she ain't gonna make it</pre>
Ain't got no time for a minute is a great line.
<pre>yeah , i'm just tryin to catch me on the street
and get caught up in my pocket full of weed
you think it's time for me to feel the breeze
i'ma put your ass in a pocket full of cheese</pre>
As my brother pointed out, I could probably create a gangster slang generator out of this.
<h2>Comments</h2>
<strong>D-Prime has issues with women</strong>

At the start of this post I said I hadn't made any effort at censorship.  This was a lie.  If left to its own devices, D-Prime writes some fairly...  coarse lyrics, often completely out of nowhere.
<pre>there's a place in my heart , i promise it will never end
but it's so hard for me to be the world's best friend
fuck that bitch , i love it when you suck my dick
i'm bout to get caught up in the heart of a chick</pre>
The reason this happened is because these kind of lines are very common in the lyrics the model was trained on.  Indeed, in the process of building this model I have read some absurdly sexist verses.  This was actually causing  problems, D-Prime literally couldn't go a verse without throwing in either some sappy line to an unnamed girl (out of nowhere), or a borderline verbal sexual assault, or indeed both within the space of two lines.

D-Prime also doesn't have a memory that lasts longer than the last two lines, which made it easy for it to get confused about what role it was supposed to be playing, for example :
<pre>ain't none of these niggas tryin to fuck with me
i need a bitch , just give me some pussy
and you know that i've been all over this globe
cause back in the days when i was sellin dope
and that's how we do it all over this world
but you should know , i wanna be your girl</pre>
For this reason I took the difficult decision of completely banning D-Prime from using certain words, which helped a lot.  This is a good illustration of how machine learning algorithms like this rarely act how you want them to without some supervision (see : <a href="http://uk.businessinsider.com/microsoft-deletes-racist-genocidal-tweets-from-ai-chatbot-tay-2016-3">Tay Tweets</a>).

<strong>Creativity</strong>

A couple of people I showed this to have commented on this, so I will be the first to say that D-Prime is not creative in the way that we usually think about creativity.  D-Prime is trying very hard to <em>exactly reproduce </em>the language of the rap it has already read.  Any originality in the output of the algorithm is due to lack of understanding of the language.  Generating text in this manner is an interesting way of showing what the model has learned about text, but it is a bit of a parlour trick really.  The model doesn't actually understand what words mean in relation to the real world, just how the grammar of how words relate to one another in a sequence.
<h2><strong>Final words</strong></h2>
To bring the post full circle, for the hip hop fans out there, here is the most 'knowing' rhyme that D-Prime generated.
<pre>you will end up in new york state of mind
cause it's all this money and the world is mine</pre>
There are two references to <em>Illmatic</em> in this rhyme.  Coincidence?  Almost certainly, but still an extremely awesome one.  Now would be a fitting time to remind everyone how much better humans are at rap lyrics (<a href="http://genius.com/Nas-lifes-a-bitch-lyrics">link to lyrics</a>).

https://www.youtube.com/watch?v=HEwSfbE9IXc&feature=youtu.be&t=20

Many thanks to my hip hop head brother Niall for his enthusiasm as the model progressed, for some suggestions and jokes on this blog post - and for generally being a straight up G.  Play us out D.
<pre>this the type of shit that i'm lookin for bro
and when it come to my hood , they call me solo
cause i came a long way from the danger zone
don't be hating on me when i'm in my dome
i'll be right back in the heart of a ghost
i'm that nigga from north , east to west coast
yeah , cause you know we came to party tonight
so throw your hands up in the air if it's alright
we can go and get a piece of that fence
you need to slow down , it don't make no sense
i grab the mic and a pocket full of beats
now let me show you how to make ends meet
cause i know that you was down with me today
but now it's time to come and get it ok
don't want no money , i gotta make a mil
everytime you see me in the back of my grill
i'm just tryin to take a look at this ho
but when i hit the flo , we make it glow
yo , i remember when we used to do shows
been around the world and that's just how it goes</pre>
