M32=/Users/bazyar/src/Merlin32_v1.1/MacOs/Merlin32
CP2=/Users/bazyar/src/cp2_1.0.5_osx-x64_sc/cp2

all: mbtest.po mbtest gstest img

mbtest.po:
	$(CP2) cdi mbtest.po 800k ProDOS

gstest: gstest.a65
	$(M32) -V include gstest.a65

mbtest: mbtest.a65
	$(M32) -V include mbtest.a65

img: gstest mbtest mbtest.po
	$(CP2) add -f mbtest.po gstest
	$(CP2) add -f mbtest.po mbtest
	$(CP2) set-attr mbtest.po type=bin,aux=0x0800 gstest
	$(CP2) set-attr mbtest.po type=bin,aux=0x0800 mbtest

