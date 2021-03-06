# Mon pops
A grnet mon plugin which can visualize the Points of presence
for grnet.

Runs under:
[mon.grnet.gr/pops/](https://mon.grnet.gr/pops/) and uses data served from
[mon.grnet.gr/api/pops/](https://mon.grnet.gr/api/pops/) to show all Points of
Presence on a map.


## Installation
- clone mon_pops and put the into mon.
- Add `pops` in installed apps
- run `./manage.py collectstatic`
- Add `url(r'^pops/', include('pops.urls')),` into patterns under grnet/urls.py
- Add `url(r'^pops/', include('pops.urls.api_urls')),` into patterns under api/urls.py


## Api
The json can return the following variables:

- pins, an array of pins that will be shown in the map
- url, the url of the page
- coords, the limits of the interestin area in the map
- links, the links from one pin to another

### Pins
- name, the name of the pin. Required.
- geolat, geolng, the coords of the point. Required.
- location, if the point is a kind of institution. Required.
- image, if an image should be used instead of the pin
- url, the url that is linked with the pin, if the pin is not a child

### Coords
The limits of the interesting area in the map:

- minlat, minimum latitude
- minlng, minimum longitude
- maxlat, maximum latitude
- maxlng, maximum longitude


### Links

- fmloc, from location
- toloc, to location

- fmto, from, to

- fmlat, from latitude
- fmlng, from longitude
- tolat, to latitude
- tolng, from logintude

- fmcity, from city
- tocity, to city

- mrtg_tag, tag
- mrtg_indexuri, uri

- fmnd, from node
- tond, to node

- capacity

- mrtg_type
- mrtg_dayuri, uri to diagram

- toif, to interface
- fromif, from interface

- mrtg_thumb, thumbnail of the diagram(?)


### The view used to create peer_ifces
	CREATE VIEW `peer_ifces` AS (
	    select
	        `node`.`name` AS `name`,
	        `b`.`ifDescr` AS `interface`,
	        `db_members`.`location`.`location_id` AS `grnet_device_location`,
	        `db_members`.`peer_site`.`site_id` AS `site`,
	        `db_members`.`peer_site`.`peer_site_location` AS `site_location`,
	        `td`.`member_id` AS peer
	    from (
	        (
	            (
	                (
	                    (
	                        (
	                            (
	                                (
	                                    `observium`.`ports_stack` join `observium`.`ports` `a` on (
	                                        (
	                                            (
	                                                `a`.`ifIndex` = `observium`.`ports_stack`.`port_id_high`
	                                            ) and (
	                                                `a`.`device_id` = `observium`.`ports_stack`.`device_id`
	                                            )
	                                        )
	                                    )
	                                ) join `observium`.`ports` `b` on (
	                                    (
	                                        (
	                                            `b`.`ifIndex` = `observium`.`ports_stack`.`port_id_low`
	                                        ) and (
	                                            `b`.`device_id` = `observium`.`ports_stack`.`device_id`
	                                        )
	                                    )
	                                )
	                            ) join `grnetdb`.`tagged_ifce` `ti` on (
	                                (
	                                    `ti`.`ifce_id` = `b`.`port_id`
	                                )
	                            )
	                        ) join `grnetdb`.`tagged_domain` `td` on (
	                            (
	                                `ti`.`tagged_domain_id` = `td`.`domain_id`
	                            )
	                        )
	                    ) join `grnetdb`.`node` on (
	                        (
	                            `node`.`node_id` = `observium`.`ports_stack`.`device_id`
	                        )
	                    )
	                ) left join `db_members`.`location` on (
	                    (
	                        `db_members`.`location`.`location_id` = `node`.`location_id`
	                    )
	                )
	            ) left join `db_members`.`peer_site` on (
	                (
	                    `ti`.`site_id` = `db_members`.`peer_site`.`site_id`
	                )
	            )
	        ) left join db_members.peer on (
	            td.member_id = db_members.peer.peer_id
	        )
	    ) where (
	        (
	            `b`.`ifDescr` is not null
	        ) and (
	            `a`.`ifDescr` is not null
	        ) and (
	            `ti`.`tagged_domain_id` <> 100
	        )
	    ) group by `b`.`port_id`
	)


## User manual
Το πρώτο πράγμα που βλέπουμε έιναι ένας χάρτης με πολείς στις οποίες υπάρχουν
σημεία παρουσίας της ΕΔΕΤ. Αν πατήσουμε σε κάποιο απο αυτά τότε βλέπουμε την
πόλη λεπτομερώς, με τα σημεία παρουσίας στην πόλη αυτή.

Αν πατήσουμε πάνω σε κάποιο σημείο παρουσίας, στο αριστερό σημείο της σελίδας
εμφανίζονται κάποιες πληροφορίες:

- Όνομα τοποθεσίας
- πληροφορίες για τον ιδιοκτήτη της τοποθεσίας αυτής
- συντεταγμένες

Επιπλέον εμφανίζονται πιθανώς και άλλες δυο πηγές πληροφορίας:

1. In this Location:
Στην περιοχή αυτή εμφανίζονται οι φορείς οι οποίοι βρίσκονται σε εκείνο το
σημείο παρουσίας. Επίσης είναι πιθανό κάτω απο κάποιο φορέα να υπάρχει το πεδίο
"through this peer" που μας δείχνει ποίος φορέας συνδέεται μέσω κάποιου άλλου.

2. Devices and Connections:
Μια λίστα με τις συσκευές που υπάρχουν εκεί, καθώς και τα interfaces που
συνδεονται με φορείς αλλά και το backbone δίκτυο της ΕΔΕΤ.
