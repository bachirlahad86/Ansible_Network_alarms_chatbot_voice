# Network_alarms_chatbot_voice
Gathering network alarms via Chatbot and voice calls <br /> 
The example used: An IOS device generating an alert when a specific interface is down. <br /> 

1. Register the **show interfaces brief** command output result:
```
 - name: show interfaces brief
      ios_command:
        commands:
          - sho ip int br
      register: int_status_raw

 - debug:
      var: int_status_raw
```

1. Using the Genie Parser as a filter to parse the output of the IOS command. The filter plugin **parse_genie.py** is available under **/filter_plugins/** folder:
```
  - name: Set Fact Genie Filter
      set_fact:
        pyatsint_status_raw: "{{ int_status_raw['stdout'][0] | parse_genie(command='show ip interface brief', os='ios') }}"
      delegate_to: localhost
```



























