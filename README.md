Laboratorium 7

Co zostało zrobione:
- Uruchomiono klaster Minikube z CNI Calico.
- Utworzono przestrzeń nazw „remote”.
- W przestrzeni „remote” uruchomiono Pod „remoteweb” z serwerem Nginx.
- Wystawiono „remoteweb” poprzez Service typu NodePort na porcie węzła Minikube 31999.
- W domyślnej przestrzeni „default” uruchomiono Pod „testpod” z obrazem Busybox (`sleep infinity`).
- Z Pod-a „testpod” sprawdzono dostęp do strony Nginx poleceniem:
  `wget --spider --timeout=1 http://remoteweb.remote.svc.cluster.local`
- Zweryfikowano działanie NodePort:
  - z węzła Minikube: `minikube ssh` + `curl http://localhost:31999`
  - z hosta: `curl http://$(minikube ip):31999` (w zależności od konfiguracji sieci hosta).

Pliki w repozytorium:

- `namespace-remote.yaml`  
  - Definiuje przestrzeń nazw „remote”, w której uruchamiane są zasoby aplikacji.  
  - Dzięki temu zasoby zadania są odseparowane od przestrzeni „default”.

- `remoteweb-pod.yaml`  
  - Definiuje Pod „remoteweb” w namespace „remote”.  
  - Używa obrazu `nginx`.  
  - Ustawia etykietę `app=remoteweb`, która jest później używana przez Service jako selector.  
  - Udostępnia port kontenera 80, na którym nasłuchuje serwer Nginx.

- `remoteweb-svc-nodeport.yaml`  
  - Definiuje Service „remoteweb” w namespace „remote”.  
  - Typ: `NodePort` – umożliwia dostęp do usługi spoza klastra.  
  - Selector: `app=remoteweb` – wskazuje na Pod o tej etykiecie.  
  - Konfiguracja portów:
    - `port: 80` – port usługi w klastrze,
    - `targetPort: 80` – port w Podzie,
    - `nodePort: 31999` – port węzła Minikube dostępny z zewnątrz.

- `testpod.yaml`  
  - Definiuje Pod „testpod” w namespace „default”.  
  - Używa obrazu `busybox`.  
  - Uruchamia komendę `["sleep", "infinity"]`, aby Pod był cały czas w stanie `Running`.  
  - Służy jako narzędzie testowe do:
    - sprawdzania DNS w klastrze,
    - testowania połączenia HTTP z usługą „remoteweb”.
