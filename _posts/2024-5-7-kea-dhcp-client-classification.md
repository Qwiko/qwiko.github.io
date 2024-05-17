---
layout: post
title: ISC DHCP and Kea DHCP Client classification!
---

Since ISC DHCP have been discontinued since [2022](https://www.isc.org/blogs/isc-dhcp-eol/){:target="\_blank"} I was tinkering with a way to recreate common operations with client classifications used in networking.

Simple example dhcpd.conf

{% highlight conf %}
class "CISCO" {
    match if (substring(option vendor-class-identifier, 0, 5) = "cisco");
    option bootfile-name "http://10.10.10.10/ios/ztp.py";
}

class "ARISTA" {
    match if (substring(option vendor-class-identifier, 0, 6) = "Arista");
    option bootfile-name "http://10.10.10.10/eos/ztp.j2";
}

subnet 10.0.0.0 netmask 255.255.255.0 {
    option routers 10.0.0.1;
    pool {
        range 10.0.0.20 10.0.0.200;
        allow members of "CISCO";
        allow members of "ARISTA";
    }
}
{% endhighlight %}

Simple example of the same functionality in kea dhcp.
{% highlight json %}
{
    "Dhcp4": {
        "interfaces-config": {
            "interfaces": [
                "eth0"
            ],
        },
        "client-classes": [
            {
                "name": "CISCO",
                "test": "substring(option[60].hex,0,5) == 'cisco'",
                "option-data": [
                    {
                        "name": "boot-file-name",
                        "code": 67,
                        "space": "dhcp4",
                        "data": "http://10.10.10.10/ios/ztp.py"
                    }
                ]
            },
            {
                "name": "ARISTA",
                "test": "substring(option[60].hex,0,6) == 'Arista' or substring(option[60].hex,0,6) == 'arista'",
                "option-data": [
                    {
                        "name": "boot-file-name",
                        "code": 67,
                        "space": "dhcp4",
                        "data": "http://10.10.10.10/eos/ztp.j2"
                    }
                ]
            },
            {
                "name": "AllGroups",
                "test": "member('CISCO') or member('ARISTA')"
            }
        ],
        "loggers": [
            {
                "name": "kea-dhcp4",
                "output_options": [
                    {
                        "output": "stdout"
                    }
                ],
                "severity": "INFO",
            },
        ],
        "subnet4": [
            {
                "subnet": "10.0.0.0/24",
                "option-data": [
                    {
                        "name": "routers",
                        "code": 3,
                        "space": "dhcp4",
                        "data": "10.0.0.1"
                    }
                ],
                "pools": [
                    {
                        "pool": "10.0.0.10 - 10.0.0.20",
                        "client-class": "AllGroups"
                    }
                ]
            }
        ]
    }
}
{% endhighlight %}

Check out my other post about: [Testing DHCP with Bats and Docker]({% link _posts/2024-5-11-testing-dhcp-with-bats-and-docker.md %}).

In a future post I will look into on commit script hooks with Kea DHCP.