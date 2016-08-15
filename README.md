#
// Icesprint.txt
// Hatsumei, Dec 24, 2015

// Jumps really quickly, allowing optimally fast traversion of ice tunnels
// There are three modes, for various environments and usage:

// Icetunnel mode, activated by being in a 2x1 area on start
//   Will detect turns, and execute them, and will automatically turn off.
//     These features are in addition to normal behavior (i.e. jumping quickly)
//     and snapping the angle on start.
//   This mode is fast and automated, but will break with up/down terrain, and
//     won't work on diagonal iceroads.

// Classic mode, activated by looking down on start
//   Will traverse iceroads, but will not attempt to turn or detect the end of
//   tunnels.  This is best used in iceroads with up/down stairs, or diagonal
//   turns.

// Freejump mode, activated by being able to fully jump on start
//   Has basically the same behavior as classic mode, except it will attempt to
//   automate turns.  This is intended to be used outside, with no tunnels at all.
//   It also does not snap.

UNSAFE(0)

#i = 0
#j = 0

#t = 0
UNSET(freejump)
UNSET(classic)
#ypos = %YPOS%

IF((%PITCH% >= 20) && (%PITCH% <= 90))
    LOG("&f[Icesprint] &aClassic mode enabled")
    SET(classic)
    $$<_snap.txt>
ENDIF

DO;
    KEYDOWN(forward)
    SPRINT()

    #hitxdist = %HITX% - %XPOS%
    #hitzdist = %HITZ% - %ZPOS%

    #hitydist = %HITY% - %YPOS%

    // If pressing forward/back buttons, terminate
    IF( (("%KEY_W%" = "True") || ("%KEY_S%" = "True")) && (%GUI% = "NONE") )
        BREAK()

    // Maintain a full hunger level
    ELSEIF(%HUNGER% <= 14)
        #oldpitch = %PITCH%
        LOOK(+0,0)
        DO(80); // Try eating for a max of 4 seconds
            IF(%HUNGER% > 14)
                BREAK()
            ENDIF

            PICK("baked_potato")
            KEY(use)
            KEYDOWN(forward)
            SPRINT()
            WAIT(1t)
        LOOP
        LOOK(+0,%#oldpitch%)

    // If there's a wall in front of us, attempt to turn (Except on classic mode)
    ELSEIF(((#hitxdist = 1) || (#hitxdist = -1) || (#hitzdist = 1) || (#hitzdist = -1)) && ((%#hitydist% = 0) || (%#hitydist% = 1)) && (!classic))
        LOOK(+90,+0)
        KEYDOWN(forward)
        SPRINT()
        WAIT(2t)

        #hitxdist = %HITX% - %XPOS%
        #hitzdist = %HITZ% - %ZPOS%

        #hitydist = %HITY% - %YPOS%

        IF(((#hitxdist = 1) || (#hitxdist = -1) || (#hitzdist = 1) || (#hitzdist = -1)) && ((%#hitydist% = 0) || (%#hitydist% = 1)))
            LOOK(+180,+0)
            KEYDOWN(forward)
            SPRINT()
            WAIT(2t)

            #hitxdist = %HITX% - %XPOS%
            #hitzdist = %HITZ% - %ZPOS%

            #hitydist = %HITY% - %YPOS%

            IF(((#hitxdist = 1) || (#hitxdist = -1) || (#hitzdist = 1) || (#hitzdist = -1)) && ((%#hitydist% = 0) || (%#hitydist% = 1)))
                LOG("&f[Icesprint] &cCannot turn, a dead-end maybe?")
                BREAK
            ELSEIF
                KEYDOWN(left)
                KEYDOWN(forward)
                SPRINT()
                WAIT(1t);
                KEYUP(left)
            ENDIF
        ELSEIF
            KEYDOWN(right)
            KEYDOWN(forward)
            SPRINT()
            WAIT(1t);
            KEYUP(right)
        ENDIF

    // Stops when a freejump is detected, except on freejump or classic modes
    ELSEIF((%YPOS% != %#ypos%) && (!freejump) && (!classic))
        LOG("&f[Icesprint] &cReached end of the iceroad, stopping")
        BREAK()

    // Jump in the optimal timing pattern
    ELSE
        IF(#i = 0)
            KEYDOWN(jump)
            #ypos = %YPOS%
            IF(#j = 0)
                DO(3)
                    KEYDOWN(forward)
                    SPRINT()
                    WAIT(1t)
                LOOP
                #j = 1
            ELSEIF(#j = 1)
                DO(4)
                    KEYDOWN(forward)
                    SPRINT()
                    WAIT(1t)
                LOOP
                #j = 2
            ELSEIF(#j = 2)
                DO(3)
                    KEYDOWN(forward)
                    SPRINT()
                    WAIT(1t)
                LOOP
                #j = 0
            ENDIF
        ENDIF
        #i = %#i% + 1
        IF(#i >= 2)
            KEYUP(jump)
            #i = 0
            WAIT(1t)
            KEYDOWN(forward)
            SPRINT()
        ENDIF
    ENDIF

    // If it's the beginning moment and a freejump happens,
    // prevent freejumps from terminating the script.
    IF((%#t% = 0) && (!classic))
        IF(%YPOS% != %#ypos%)
            LOG("&f[Icesprint] &aFreemode enabled")
            SET(freejump)
        ELSE
            LOG("&f[Icesprint] &aIcetunnel mode enabled")
            $$<_snap.txt>
        ENDIF
        INC(#t)
    ENDIF

LOOP

KEYUP(forward)
KEYUP(jump)

ENDUNSAFE()
