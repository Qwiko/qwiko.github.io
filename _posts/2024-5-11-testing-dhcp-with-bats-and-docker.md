---
layout: post
title: Testing DHCP with Bats and Docker!
---

[Official KEA Docker](https://gitlab.isc.org/isc-projects/kea-docker)

[Git Repo](https://github.com/Qwiko/dhcp-bats-docker)

bats test-file dhcp-docker-tests.bats
{% highlight json %}
function docker_udhcpc() {
    docker compose -f test/test-docker-compose.yaml exec -it busybox udhcpc -q -B -v -O bootfile -n -q -V $1
}

@test "test vendor: unknown" {
    run docker_udhcpc unknown_vendor
    [ "$status" -eq 1 ]
}

@test "test vendor: ciscopnp" {
    run docker_udhcpc ciscopnp
    [ "$status" -eq 0 ]
    [ "${lines[-1]}" = "http://10.10.10.10/ios/ztp.py" ]
}

@test "test vendor: Arista" {
    run docker_udhcpc Arista
    [ "$status" -eq 0 ]
    [ "${lines[-1]}" = "http://10.10.10.10/eos/ztp.j2" ]
}

@test "test vendor: arista" {
    run docker_udhcpc arista
    [ "$status" -eq 0 ]
    [ "${lines[-1]}" = "http://10.10.10.10/eos/ztp.j2" ]
}
{% endhighlight %}

Running the tests
{% highlight bash %}
$ bats test
dhcp-docker-tests.bats
 ✓ test vendor: unknown
 ✓ test vendor: ciscopnp
 ✓ test vendor: Arista
 ✓ test vendor: arista
{% endhighlight %}