# Finding The "Yellow Plane"

![](/articles/media/yellow-plane-challenge-tweet.png)

Hello everyone, recently I came across a tweet by [Joe Helle](https://twitter.com/joehelle) which was a simple OSINT challenge. The challenge was to identify the exact spot from where the photograph is taken.

The challenge picture in full resolution can be seen below:
![](/articles/media/yellow-plane-thumbnail.jpeg)

For those who have seen it, a simple "Guardians of the Galaxy Yello Plane" google search would have solved the mystery, but many of us(like me), don't know anything about it. So for me, reverse engineering the image was the only way.

## Investigation
Let's first try to find out what all information is available to us in this image.
1. A yellow plane (In animation world, a "ship")
2. A spherical structure (behind the ship)
3. If you look close to the left of the image, there is some shop/store there, of which, the only visible letters are:
```
CON**CTIONS
EATE****
```
![](/articles/media/connections-cafe-hint.png)

Based on the width of the letters and the pole that has shawdowed the letters behind it, there could be two letters behind that pole.
So most probably, the word would be **CONNECTIONS**

So now the updated query looks like:
```
CONNECTIONS
EATE****
```

Upon searching the word "Connections eate" on google, this is what we get:
![](/articles/media/connection-cafe-google.png)
Based on the suggestions, it would be safe to say that the word is **CONNECTIONS EATERY** which is a café in Disney World.

An image of the front of the café can be seen on this [Disney World Page](https://disneyworld.disney.go.com/dining/epcot/connections-eatery/):
![](/articles/media/connections-cafe-sphere.png)

Even though this picture has both the text and that spherical structure, there is a problem here. In the above picture, the word "EATERY" is written on the right side below the word "CONNECTION" but the same term is on the left side in the challenge picture. So we need to look at more pictures of this café.

[This article](https://wdwnt.com/2022/04/plants-and-another-sign-installed-outside-connections-cafe-and-eatery-at-epcot/) shows numerous pictures of the surroundings of the café, here is one picture that caught my eye:
![](/articles/media/connections-cafe-sideview.png)
This image contained both Connections Café logo with grass walls as well as the spherical structure in the expected direction and expected distance.

Now it is confirmed that we are looking at the correct outlet and at the correct spot.

To find the exact location where this photograph was taken, we need to perception analysis. Based on the view shown in the challenge picture, this appears to be the perspective-
* The person is standing few meters from the front of the cafe but far from the sphere.
* Both the spherical structure and cafe sign board lie on the other side of the bridge.
* There is a building on the right side of the viewer with pointed edges.

Now lets search "Connections Eatery Epcot" on Google Earth. Based on the perspective, this appears to be the location from where the photo was taken.
![](/articles/media/google-earth-topview.png)

Upon checking the street view, this view is revealed.
![](/articles/media/google-earth-streetview.png)

The location looks promising, also the spherical structure is also at the expected angle, but neither the Connections Cafe is visible nor the Yellow Ship. Maybe we are looking at an older picture.

Upon googling the words written there in blue, "Universe of Energy", we discover [this](https://en.wikipedia.org/wiki/Universe_of_Energy) wikipedia page which shows the history of this building. As can be seen in the image below, it says that it has now been replaced with Guardians of the Galaxy: Cosmic Rewind.
![](/articles/media/universe-of-energy-wikipedia.png)

The image of that yellow ship can be seen on the wikipedia page of [Guardians of the Galaxy: Cosmic Rewind](https://en.wikipedia.org/wiki/Guardians_of_the_Galaxy:_Cosmic_Rewind).
![](/articles/media/the-yellow-ship.png)


And the challenge is now solved!

Thanks for reading! :) 