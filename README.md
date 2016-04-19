
# WebScale infrastructure with Kubernetes and Flannel

## Talk

### Abstract [ITA]

La capacità di rispondere in poche frazioni di secondo alle richieste degli utenti - indipendentemente dal loro numero - è un fattore determinante per il successo dei servizi sul web. Secondo Amazon,  bastano 100 millisecondi di latenza nella risposta per generare una perdita economica di circa l'1% sul
fatturato [1]. In base alle statistiche di Google AdWords, inoltre, il 2015 ha sancito l’ufficiale superamento del numero di interazioni mobile rispetto a quelle desktop [2], con la conseguente riduzione della durata media delle sessioni di navigazione web.
In uno scenario di questo tipo, la razionalizzazione dell’utilizzo delle risorse hardware e la capacità di scalare rispetto al numero di utenti sono fattori determinanti per il successo del business.
In questo talk racconteremo la nostra esperienza di migrazione di soluzioni e-commerce di tipo enterprise in Magento da un’architettura basata su VM tradizionali ad una di tipo software-defined basata su Kubernetes, Flannel e Docker. Discuteremo, quindi, delle reali difficoltà da noi incontrate nel porting su container di soluzioni in produzione e daremo evidenza di come, alla fine di questo lungo viaggio, i nostri sforzi siano stati concretamente premiati dall’aumento di resilienza, affidabilità e automazione della soluzione finale.
A supporto della conversazione, mostreremo i risultati dei benchmark da noi condotti per valutare la scalabilità della nuova architettura presentando delle evidenze delle reali capacità di Kubernetes come strumento di orchestrazione di servizi erogati in Docker container.
Concluderemo l’intervento presentando il nostro progetto di distribuzione geografica dei nodi master di Kubernetes facendo uso di reti SD-WAN per garantire performance e continuità di servizio della soluzione.

[1] http://radar.oreilly.com/2008/08/radar-theme-web-ops.html <br/>
[2] http://searchengineland.com/its-official-google-says-more-searches-now-on-mobile-than-on-desktop-220369

### Slides

http://www.slideshare.net/purpleocean/web-scale-infrastructures-with-kubernetes-and-flannel

## Source Code

These files can be used to

1. Install a Kubernetes and Flannel cluster on DigitalOcean.
2. Create a `PersistentVolume` to setup a shared NFS storage backend for Kubernetes.
3. Run a LAMP stack on Kubernetes leveraging the concepts of `HorizontalPodAutoscaler` and `ReplicationController`
   to guarantee the web applications auto-scaling and self-healing.

### Requirements

- Vagrant >= 1.8
- Ansible >= 2.0

### Setup

1. Get the submodules

  ```bash
  git submodule init
  git submodule update
  ```

2. Install the vagrant-digitalocean provider

  ```bash
  vagrant plugin install vagrant-digitalocean
  ```

3. Create a new file called `env.sh` into the `scripts/` directory with the
following content

  ```bash
  export DIGITAL_OCEAN_TOKEN=<your digital_ocean API key>
  export NUM_MASTERS=1 # the number of Kubernetes master nodes
  export NUM_MINIONS=2 # the number of Kubernetes minions nodes
  ```

4. Create and provision the Kubernetes droplets with

  ```bash
  source env.sh
  vagrant up --provider=digital_ocean
  ```

### Run the sample on Kubernetes

1. Create the persistent volume and the persistent volume claim for your NFS server.

  ```bash
  vagrant ssh kube-master-1
  kubectl create -f /vagrant/kubernetes-files/web-storage-pv.yaml
  kubectl create -f /vagrant/kubernetes-files/web-storage-pvc.yaml
  ```

2. Create the web-application replication controller and service

  ```bash
  vagrant ssh kube-master-1
  kubectl create -f /vagrant/kubernetes-files/web-frontend-rc.yaml
  kubectl create -f /vagrant/kubernetes-files/web-frontend-svc.yaml
  ```

3. Scale manually the web-frontend

  ```bash
  vagrant ssh kube-master-1
  kubectl scale --replicas=10 rc/web-frontend
  ```

4. Set-up Horizontal Pod Autoscaling

  ```bash
  vagrant ssh kube-master-1
  kubectl create -f /vagrant/kubernetes-files/web-frontend-hpa.yaml
  ```

## Acknowledge

The present work has been developed thanks to the great support of every single
person working at [PurpleOcean](www.purpleocean.it). <br/>
<center>
  <h3>Thank you!<h3>
  <img src="http://www.purpleocean.it/wp-content/uploads/2016/04/purpleocean-logo.png" />
</center>
