---
title: Motor Control via ESC Using Arduino and Clodiuno
tags: arduino clodiuno clojure
---

This post is partly for safe keeping and partly for providing more
documentation for [Clodiuno](/clodiuno.html). Basically
[ESCs](http://en.wikipedia.org/wiki/Electronic_speed_control) are
[PWM](http://en.wikipedia.org/wiki/Pulse-width_modulation) controllers
for electric motors (or more simply put they allow you to alter the
speed of an electric motor). 

Hooking them up to an arduino board is pretty simple, ground on the ESC
(usually the black cable) goes to ground on the arduino, while signal
(usually white or yellow) goes to a digital pin, do not connect +5V
(usually the red cable) to anywhere. You can treat an ESC just like a
servo. They take values between 0 and 180, values between 0 and 90 will
turn the motor in reverse while values between 90 to 180 will turn the
motor forward. Most ESCs requires you to arm them by sending the minimum
throttle signal (usually 90), most will also let you know that they are
armed by beeping or blinking.

Following snippet is what I use for quickly testing various PWM values,

     (ns esc-test.core
       (:use clodiuno.core)
       (:use clodiuno.firmata)
       (:import (javax.swing JFrame JPanel JSlider JTextField JLabel)
                (java.awt BorderLayout)
                (javax.swing.event ChangeListener)))

     (def esc-pin 10)

     (defn slider [pwm]
       (doto (JSlider. JSlider/HORIZONTAL 0 180 @pwm)
         (.addChangeListener 
          (proxy [ChangeListener] [] 
            (stateChanged [e] (dosync (ref-set pwm (.getValue (.getSource e)))))))
         (.setMajorTickSpacing 30)
         (.setMinorTickSpacing 10)
         (.setPaintTicks true)
         (.setPaintLabels true)))

     (defn panel [pwm]
       (let [slider (slider pwm)
             field (JTextField. (str @pwm) 3)
             field-panel (JPanel.)]
         (add-watch pwm "text-field" (fn [k r o n] (.setText field (str n))))
         (doto field-panel
           (.add (JLabel. "PWM: "))
           (.add field))
         (doto (JPanel. (BorderLayout.))
           (.add slider BorderLayout/NORTH)
           (.add field-panel BorderLayout/SOUTH))))

     (defn esc [pwm]
       (let [board (arduino :firmata "/dev/tty.usbserial-A600aeCj")]
         ;;allow board to boot
         (Thread/sleep 5000)
         ;;attach ESC
         (pin-mode board esc-pin SERVO)
         (analog-write board esc-pin 90)
         (add-watch pwm "esc-set" (fn [k r o n] (analog-write board esc-pin n)))))

     (defn frame []
       (let [pwm (ref 90)
             panel (panel pwm)]
         (esc pwm)
         (doto (JFrame. "ESC Tester")
           (.add panel)
           (.pack )
           (.setSize 250 115)
           (.setVisible true))))

     (frame)
