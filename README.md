# Restore
Need to perform restore drill activity
provider "google" {

  project     = rapid-domain-416914
  region      = "us-west4-b"  # Replace with your desired region
}

data "google_compute_snapshot" "source_snapshot" {
  name = snapshot-drill
  project = rapid-domain-416914

}

locals {
  source_project = My First Project
  target_project = My First Project
  source_disk_snapshot = snapshot-drill
  #data_disk_snapshots = var.datadisk_snapshots
}

resource "google_compute_disk" "source_disk" {
  name  = "restoredrill"    #name of the disk in target project
  type  = "Balanced persistent disk"
  zone  = us-west4-b
#  size  = var.size
  snapshot = data.google_compute_snapshot.source_snapshot.self_link
  project = local.target_project
}


resource "google_compute_instance" "vm_instance" {
  name         = restoredrill
  machine_type = e2-medium
  zone         = us-west4-b
  project = local.target_project
  boot_disk {
      source = google_compute_disk.source_disk.self_link
  }

 network_interface {
    network = var.network
	subnetwork  = var.subnetwork
    //access_config {
      // The "0.0.0.0/0" here allows access from any IP. Consider restricting access for security.
      //nat_ip = google_compute_address.static_ip.address
    //}
  }

  metadata = {
    // You can add any additional metadata here if needed.
}

  metadata_startup_script = "#!/bin/bash\n\n# Add your custom startup script here if needed.\n"

  service_account {
    scopes = ["userinfo-email", "compute-ro", "storage-ro"]
  }
    depends_on = [google_compute_disk.source_disk]
  // You may add other configurations like service accounts, labels, etc. as needed.
}
