# Basic Customisations.

## IPL

firstipl.sh 
awsckmap devmap.txt
awsstart --map devmap.txt
x3270 -port 3270 mstcon@localhost &
x3270 -port 3270 tso@localhost &
ipl 0a80 parm 0A82CS

Respond to Master Console Prompts on first IPL
