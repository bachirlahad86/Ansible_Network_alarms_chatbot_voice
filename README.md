# Network_alarms_chatbot_voice
Gathering network alarms via Chatbot and voice calls. <br /> 
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


3. Sending the info about the down interfaces to a Chatbot Discord Channel of your choice. The corresponding channel username and key should be used:
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

4. Sending the same info about the down interfaces to the text to speech engine. The engine used in our test playbook is the **Voice RSS** tts. The corresponding license key should be used:
```
 - name: Send Show Interface Status (DOWN/DOWN) to tts Voice RSS
      uri:
        url: "http://api.voicerss.org/?key=25f6c2d1c4a447d7ab0f2dcd80&hl=en-us&c=MP3&src=The%20interface%20{{ item.key | regex_replace('/','_') }}%20on%20{{ ansible_facts['net_hostname'] }}%20is%20{{ item.value.status | regex_replace(' ','%20') }}"
        method: GET
        return_content: yes
        validate_certs: no
      delegate_to: localhost
      register: speech_raw
      loop: "{{ pyatsint_status_raw.interface | dict2items }}"
      when: item.value.status == "administratively down"
```

5. Copy the returned audio files to a folder ready to play:

```
   - name: Copy to file ready to play
      copy:
        content: |
          {{ item.content }}
        dest: "{{ item.item.key | regex_replace('/','_') }}on{{ ansible_facts['net_hostname'] }}.MP3"
      loop: "{{ speech_raw.results }}"
      when: item.content_type is defined

```


## Playbook execution
Run the Ansible playbook:<br /> **ansible-playbook -i invent.yml play_discord_voice.yml --ask-vault-pass** <br />



## Rewards
- Genie and Ansible authors and developers




















