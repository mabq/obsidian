# Omarchy

Super nice, but it has a lot of stuff I don't need and misses many things I do need.


**About the things it has that I don't need**

- Much of the Bash code in Omarchy is aimed to provide menus and amenities that any user would need to interact with the system. This is stuff that I actually want to know how it works anyway, so I don't need to be abstracted away.


**About the things it does not have that I need**

- Some of those parts are easy to add, yet those changes will need to be replicated manually on every machine where I install Omarchy on. This is a deal breaker already.
- Some other parts are not so easy to add, for example:
	- Omarchy uses Bash, I prefer Zsh. I would need to recreate everthing Omarchy expects to find in its Bash configuration into Zsh. Updates could add/change stuff without I knowing about.
	- I use a different Neovim configuration, which expects some packages to be available on the system, packages that Omarchy does not install by default and that I would need to install manually on every machine.


**Conclusion**

My efford would be focused on adjusting Omarchy to my preferences, instead of building a system designed for my preferences.

Use Omarchy as inspiration and learning source instead.