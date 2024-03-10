## graduated-room-lights

![Example Roof](https://github.com/philmale/graduated-room-lights/blob/main/graduatedLights.jpg?raw=true)


Script to set a graduated transition from one colour to another across a group of Philips Hue lights (should work with any light that can understand HS parameters for colour), you can specify a light group or a list of explicit lights to transition the colours across, set the starting and/or ending hue, saturation and brightness - if anything is missing an appropriate random value will be selected - and you can choose a number of different ways of stepping across the HS colour space (complementary, analogous, close, subtle, triadic, tetradic, random).
Looks great in rooms where there are several Hue ceiling spots, but will work across any number of lights in any group. 
## Example
An example of how to call it.
A script to colour from one random colour to another across a room with four ceiling lamps and two floor lamps, with the lights specifically named so they are turned on in that order, you could also just put in light.front_hall (or whatever your room group is).

```
alias: Front Hall Rainbow
sequence:
  - service: script.graduated_room_lights
    data:
      room:
        - light.fhlpr
        - light.fhr1
        - light.fhr2
        - light.fhl1
        - light.fhl2
        - light.fhlpl
      hue_mode: complementary
      from:
        hue: 0
        saturation: random
        brightness: 255
      to:
        hue: 240
        saturation: random
        brightness: 255
      hue_shortest: false
      transition: 2
mode: restart
icon: mdi:palette-outline
```

If you don't specify from or to hue or saturation the script will pick random values, so by not specifying the from.hue or the to.hue settings you'll get a random rainbow every time from one colour to another.

The hue_shortest parameter tells the script to pick the shortest path between two points on the colour wheel to create the graduation or not (so any two points on the wheel will have a short route between them or a long route around the wheel between them - this tells the algorithm which way to go!).

You can also pass an exclude parameter that is a comma separated list of lights to exclude from the calculation - so if you have a room with a several lights in it and you want to exclude two of them you can set room to light.room_name and exclude to (say) light.light_3, light.light_4

The transition parameter will set the transition to that many seconds, so in my example above it takes two seconds to colour the room.

## Try It
If you want to play with this logic, here is a chunk of code you can copy and paste into the HA template editor to mess around with the script parameters and see how it all works, the script is tagged on the end here so you can just lift this code directly into the template editor:

```
{# Cut and paste me into the Developer Tools Template Editor in HA to view the JSON scene that would be applied #}
{# Here you can try different parameters to the script #}

{# I've put some defaults in, you will want to play with your own light names #}
{# The last value set for any variable will be the prevailing one in the template editor... obviously #}

{# Setting room to a hue group in whatever order they appear in the group #}
{% set room = 'light.bedroom' %}
{# Setting room to an explicit list #}
{% set room = ['light.bec1', 'light.bec2', 'light.bec3', 'light.bec4', 'light.bec5'] %}

{# Removing two lights from the scene as a string #}
{% set exclude = 'light.bec2,light.bec3' %}       
{# Removing two lights from the scene as an array #}
{% set exclude = ['light.bec2','light.bec3'] %}

{# Set the hue_mode - the way in which adjacent colours are selected #}
{% set hue_mode = 'subtle' %}
{% set hue_mode = 'analogous' %}
{% set hue_mode = 'tetradic' %}

{# Set the route around the HS colour wheel - directly between two colours, or the long way around #}
{% set hue_shortest = false %}

{# Set explicit values for from colour #}
{% set from = namespace(hue=60,saturation=45,brightness=255) %}
{# ... or let the script pick some random starting values #} 
{% set from = namespace(hue=130) %}

{# Same mechanics work for the to colour #}
{% set to = namespace(hue=80,saturation=100,brightness=155) %}


{# Script mechanics below - XXX DO NOT EDIT BELOW (well unless you have a suggestion!) #}

        {# Build a graduated room #}

        {%- macro h_complementary(c) -%}
          {{- ((c + 180) % 360) | round(3) -}}
        {%- endmacro -%}

        {%- macro h_random_analogous(c) -%}
          {{- ((c + ([-30, 30] | random)) % 360) | round(3) -}}
        {%- endmacro -%}

        {%- macro h_random_analogous_close(c) -%}
          {{- ((c + ([-15, 15] | random)) % 360) | round(3) -}}
        {%- endmacro -%}

        {%- macro h_random_analogous_subtle(c) -%}
          {{- ((c + ([-7.5, 7.5] | random)) % 360) | round(3) -}}
        {%- endmacro -%}

        {%- macro h_random_triadic(c) -%}
          {{- ((c + 120 * ([1, 2] | random)) % 360) | round(3) -}}
        {%- endmacro -%}

        {%- macro h_random_tetradic(c) -%}
          {{- ((c + 90 * ([1, 2, 3] | random)) % 360) | round(3) -}}
        {%- endmacro -%}

        {%- macro h_select_alt(h_mode,c) -%}
          {%- if h_mode == "complementary" -%}
            {{- h_complementary(c)|float -}}
          {%- elif h_mode == "analogous" -%}
            {{- h_random_analogous(c)|float -}}
          {%- elif h_mode == "close" -%}
            {{- h_random_analogous_close(c)|float -}}
          {%- elif h_mode == "subtle" -%}
            {{- h_random_analogous_subtle(c)|float -}}
          {%- elif h_mode == "triadic" -%}
            {{- h_random_triadic(c)|float -}}
          {%- elif h_mode == "tetradic" -%}
            {{- h_random_tetradic(c)|float -}}
          {%- elif h_mode == "random" -%}
            {{- pick_hue()|float -}}
          {%- else -%}
            {{- c -}}
          {%- endif -%}
        {%- endmacro -%}

        {%- macro pick_hue() -%}
        {{- (range(0,36000) | random) / 100 -}}
        {%- endmacro -%}

        {%- macro pick_sat() -%}
        {{- (range(4000,10000) | random) / 100 -}}
        {%- endmacro -%}

        {%- macro pick_br() -%}
        {{- (range(20000,25500) | random) / 100 -}}
        {%- endmacro -%}

        {%- macro next_hue() -%}
        {{ c_set.hue }}
        {%- set c_set.hue = ((c_set.hue + steps.hue) % 360) -%}
        {%- endmacro -%}

        {%- macro next_sat() -%}
        {{ c_set.sat }}
        {%- set c_set.sat = (c_set.sat + steps.sat) -%}
        {%- endmacro -%}

        {%- macro next_br() -%}
        {{ c_set.br }}
        {%- set c_set.br = (c_set.br + steps.br) -%}
        {%- endmacro -%}

        {%- macro pick_hs(lights) -%}
        {%- for light in lights -%}
          "{{- light -}}": {"state":"on","hs_color": [ {{- next_hue()|round(3) -}}, {{- next_sat()|round(3) -}} ],"brightness": {{- next_br()|round(3) -}}},
        {%- endfor -%}
        {%- endmacro -%}

        {%- macro breakdown_group(group) -%}
          {%- if state_attr(group, 'is_hue_group') -%}
            {%- set entities = state_attr(group, 'lights') | map('lower') | map('regex_replace', '^(.*)', 'light.\\1') -%}
          {%- elif state_attr(group, 'entity_id') is not none -%}
            {%- set entities = state_attr(group, 'entity_id') | map('lower') | list -%}
          {%- else -%}
            {%- set entities = [group] | map('lower') -%}
          {%- endif -%}
          {{- entities | unique | join(',') -}}
        {%- endmacro -%}

        {%- macro lights(list) -%}
          {%- set t = namespace(lights = []) -%}
          {%- if list is string -%}
            {%- set t.lights = breakdown_group(list).split(',') -%}
          {%- else -%}
            {%- for l in list -%}
              {%- set t.lights = t.lights + breakdown_group(l).split(',') -%}
            {%- endfor -%}
          {%- endif -%}
          {{- t.lights | unique | join(',') -}}
        {%- endmacro -%}

        {%- macro get_lights(include, exclude) -%}
          {{- lights(include).split(',') | reject('in', lights(exclude).split(',')) | unique | join(',') -}}
        {%- endmacro -%}

        {%- macro moded_vector(s,f,t,m) -%}
          {%- set r = ((f-t)+360)%360*-1 -%}
          {%- set l = ((t-f)+360)%360 -%}
          {%- if s -%}
            {%- set d = [r|abs,l|abs]|min -%}
          {%- else -%}
            {%- set d = [r|abs,l|abs]|max -%}
          {%- endif -%}
          {%- if d == r|abs -%}
            {{ r }}
          {%- else -%}
            {{ l }}
          {%- endif -%}
        {%- endmacro -%}

        {%- set hue_mode = hue_mode if hue_mode is defined else "close" -%}
        {%- set f_hue = from.hue|float if 
          from is defined and from.hue is defined 
          and from.hue != "random" else pick_hue()|float -%}
        {%- set f_sat = from.saturation|float if 
          from is defined and from.saturation is defined 
          and from.saturation != "random" else pick_sat()|float -%}
        {%- set t_hue = to.hue|float if 
          to is defined and to.hue is defined 
          and to.hue != "random" else h_select_alt(hue_mode,f_hue)|float -%}
        {%- set t_sat = to.saturation|float if 
          to is defined and to.saturation is defined
          and to.saturation != "random" else pick_sat()|float -%}
        {%- set f_br = from.brightness|float if 
          from is defined and from.brightness is defined 
          and from.brightness != "random" else pick_br()|float -%}
        {%- set t_br = to.brightness|float if 
          to is defined and to.brightness is defined
          and to.brightness != "random" else pick_br()|float -%}
        {%- set hue_shortest = hue_shortest if 
          hue_shortest is defined else true -%}
        {%- set steps = namespace(count=0,hue=0,sat=0,br=0) -%}
        {%- set c_set = namespace(hue=f_hue,sat=f_sat,br=f_br) -%}
        {%- set exclude = exclude if exclude is defined else [] -%}
        {%- set lights = (get_lights(room,exclude)).split(',') -%}
        {%- set steps.count = (lights|length) - 1 -%}
        {%- set steps.hue = moded_vector(hue_shortest,f_hue,t_hue,360)|float /
        steps.count -%}
        {%- set steps.sat = (t_sat - f_sat) / steps.count -%}
        {%- set steps.br = (t_br - f_br) / steps.count -%}
        {{- ("{" + pick_hs(lights)[:-1] + "}") | from_json -}}

```

The DICT in the output is the scene that will be applied to the lights.
