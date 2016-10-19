---
layout: post
title: A Peacock Necklace and JavaScript
---

<img src="/photos/peacockNecklace.jpg" class="peacock" />

My daughter (Kaitlyn), prefers birthday party themes not found on store shelves; this year is no exception.  For her 6th birthday, her request is a peacock theme.  While Chuck-E-Cheese (or similar centers for youthful, celebratory pandemonium) may eliminate the need for peacock themed decorations, I still want to incorporate her favorite bird into her big day.  The result is the peacock necklace shown above. 

Were beads and jewelry techniques expressed in JavaScript, it might look something like this:

function peacockNecklace(beads) {
	var bead = [gold, mediumBlue, smallLightBlue, largeBlue, green,  bronze, purpleBlue];
	
	var ring;
	var wire;
	var beak = wire + 2 * bead[0];
	var head = wire + bead[1];
	var neck = wire + 3 * bead[2];
	var torso = wire + bead[3];
	
	var upperBody = [beak, head, ring, neck, torso];
	
	var leg = 0
	if (leg < 4) {
		leg += bead[0];
	}
	else {
		leg += bead[5] + bead[0];
	}
	
	var tailFeather = 0
	for (var tail = 0; tail < tailFeather.length; tail++) {
		if (tail < 14) {
			tailFeather += bead[4];
		}
		else {
			tailFeather += bead[5] + bead[6] + bead[5] + bead[4];
		}
	}
	
	var peacock = [upperBody.join(''), 2 * leg, 7 * tailFeather]
	
	return peacock.join('')
}
