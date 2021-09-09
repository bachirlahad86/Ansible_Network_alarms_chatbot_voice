# Network_alarms_chatbot_voice
Gathering network alarms via Chatbot and voice calls <br /> 
The example used: An IOS device generating an alert when a specific interface is down. <br /> 

## Installation prerequisites
Installing Python, Genie packages and Ansible.

## Playbook tasks

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

2. Using the Genie Parser as a filter to parse the output of the IOS command. The filter plugin **parse_genie.py** is available under **/filter_plugins/** folder:
```
  - name: Set Fact Genie Filter
      set_fact:
        pyatsint_status_raw: "{{ int_status_raw['stdout'][0] | parse_genie(command='show ip interface brief', os='ios') }}"
      delegate_to: localhost
```


2. Sending the info about the down interfaces to a Chatbot Discord Channel of your choice. The corresponding channel username and key should be used:
```
  - name: Send Show Interface Status (DOWN/DOWN) to Discord Channel
      uri:
        url: "https://discord.com/api/webhooks/823184898541486080/P1llYwJu-Rf_ZlI_Htquzs1K11k8AUCrp_NoeKy5mSBz6zvwmFy3xuIiBX"
        method: POST
        body_format: json
        status_code: 204
        return_content: no
        validate_certs: no
        body:
          username: bachir
          content: The Network Has An Alert For You from the Show Interface Brief Command on the Core
          embeds:
            - title: The following interface on {{ ansible_facts['net_hostname'] }} is DOWN / DOWN
            - description: "{{ item.key }} is {{ item.value.status }}"
      delegate_to: localhost
      loop: "{{ pyatsint_status_raw.interface | dict2items }}"
      when: item.value.status == "administratively down"
```



## Playbook execution
Run the Ansible playbook:<br /> **ansible-playbook -i invent.yml play_discord_voice.yml --ask-vault-pass** <br />



## Rewards
- Genie and Ansible authors and developers




















