---
title: "Tinkering with Philips Hue Lights"
date: 2017-11-01T17:08:09+03:00
draft: false
---

I’m a developer by day, and mostly curious the rest of the time. I recently got the Philips Hue Starter Kit 3rd Generation ([See Review Here](https://www.youtube.com/watch?v=DQYXJXybK9U)), got the [Hue Disco app](https://www.youtube.com/watch?v=WjXzdmjPMhU), and raved on my own before I was quickly bored. So I decided to see how else I could use the bulbs.

## Can I change the color on my own… 🤔
Turns out, there’s [extensive APIs and SDKS](https://www.developers.meethue.com/tools-and-sdks) for the Hue Lights.

So I quickly schemed through the Getting Started guide. Grabbed a Go SDK, and tried to cycle through the three colors I wanted (RED, GREEN, BLUE), too original.

Note that the code below assumes you’ve gotten a username and linked the hue bridge from following [this guide](https://developers.meethue.com/documentation/getting-started). This changes the color of the Hue Bulbs every 2 seconds. (Test case was done with one connected bulb)

#### hue.go
{{< highlight go >}}
package main

import (
	"time"

	hue "github.com/collinux/gohue"
)

const (
	username = "<your-username>"
)

func main() {
	bridges, _ := hue.FindBridges()
	bridge := bridges[0]
	bridge.Login(username)

	lights, _ := bridge.GetAllLights()
	for _, light := range lights {
		light.SetBrightness(50)

		light.SetColor(hue.RED)
		time.Sleep(2 * time.Second)

		light.SetColor(hue.YELLOW)
		time.Sleep(2 * time.Second)

		light.SetColor(hue.GREEN)
	}
}
{{< / highlight >}}

## HTTP Endpoint
I decided to build a tiny Go server which would sit and accept http requests, and change the lights accordingly.
{{< highlight go >}}
package main

import (
	"fmt"
	"net/http"

	hue "github.com/collinux/gohue"
	"github.com/fatih/color"
)

const (
	username   = "<username>"
	brightness = 50
)

func main() {
	bridges, _ := hue.FindBridges()
	bridge := bridges[0]
	bridge.Login(username)

	lights, err := bridge.GetAllLights()
	if err != nil {
		// go get github.com/fatih/color
		// Fatih's color lib helps with colorful stdout
		color.Red(err.Error())
	}

	http.HandleFunc("/red", redHandler(lights))
	http.HandleFunc("/green", greenHandler(lights))
	http.HandleFunc("/blue", blueHandler(lights))
	http.ListenAndServe(":8080", nil)
}

func redHandler(lights []hue.Light) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		changeHueBulbsColor(lights, hue.RED)
		fmt.Fprintf(w, "red")
	}
}

func greenHandler(lights []hue.Light) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		changeHueBulbsColor(lights, hue.GREEN)
		fmt.Fprintf(w, "green")
	}
}

func blueHandler(lights []hue.Light) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		changeHueBulbsColor(lights, hue.BLUE)
		fmt.Fprintf(w, "blue")
	}
}

func changeHueBulbsColor(lights []hue.Light, color *[2]float32) {
	for _, light := range lights {
		light.SetBrightness(50)
		light.SetColor(color)
	}
}
{{< / highlight >}}

Code could be improved 🤷‍. But in a nutshell, we have 3 endpoints:- `/red`, `/green` , `/blue` , that respectively change the Hue Bulb colors to Red, Green and Blue.

Well, that doesn’t do much, but it seemed fun. A little more tinkering coming soon. 😃
