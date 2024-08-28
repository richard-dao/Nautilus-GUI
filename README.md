# Nautilus GUI Desktop

This guide is the **MANUAL** way to get the Nautilus GUI working. Please read all the instructions carefully.

**Note:** Nautilus does update fairly frequently, so this guide may be outdated. Please check when this was last updated and compare with the Nautilus GitLab.

**Guide last updated: 8/28/2024**

## 1. Install Kubernetes and Get Added to AIEA Namespace

Nautilus is a Kubernetes hypercluster, so we interact with it using well, Kubernetes. Make sure that you have the Kubernetes command line, kubectl, installed on your device. You can follow the Nautilus quickstart guide as it goes through the installation process (the AIEA Notion also has very helpful guides on this process as well):

https://docs.nationalresearchplatform.org/userdocs/start/quickstart/

You'll also need to be added to the AIEA namespace. If your not in the AIEA lab namespace please request in the nautilus-channel on the AIEA discord. 

## 2. Creating the Persistent Volume Claims (PVC)

*Update:* As of Aug 14th, 2024, Nautilus no longer supports seaweedfs-storage-nvme for the `storageClassName`. If you have previous PVCs with seaweedfs, they are unforunately now corrupted and no longer accessible. You'll need to change over to another sotrageClassName. `rook-cephfs-pacific` is recommended.

For the Nautilus GUI, you'll need 2 PVCs, a cache and a storage. If this is your first time, you can download the files `xgl-cache.yaml` and `rook-storage.yaml` that are provided in this repository. 

Please change the name (line 4) of both files to something unique. You **cannot** have the same name PVC as someone else in the namespace.

To create the PVCs, run:

`kubectl create -f xgl-cache.yaml`
`kubectl create -f rook-storage.yaml`

To check that they were successfully created and running, you can do `kubectl get pvc` and look for the unique names of your PVC.

## 3. Get the Turn Shared Secret from Nautilus Support

*Update:* As of Aug 28th, I'm not sure if you need a turn secret anymore, but for security purposes, I think it's still best to get one.

You'll need a turn secret in order to run the Nautilus GUI. You can ask for your turn secret from Nautilus Support by asking in the general channel of their Matrix channel here: https://app.element.io/#/room/#general:matrix.nrp-nautilus.io

You may need to make a Matrix account. They'll DM you your unique turn secret so check back frequently. (**Note:** They can sometimes take a fair amount of time to respond to you or miss your message entirely. If they don't message you within a few days, feel free to ping them again)

## 4. Create Your Secret

Kubernetes Secrets are python dictionaries that hold your sensitive information as reference in the namespace. They are encrypted and allow you to lock your containers as needed. 

Please run: 

`kubectl create secret generic [my-pass] --from-literal=[my-password]=[YOUR_PASSWORD] --from-literal=[turn-secret]=[TURN_SHARED_SECRET_FROM_NAUTILUS]`

- Ensure that you use a unique name for `my-pass` as you cannot have the same Secret name as someone else in the namespace
- Replace `YOUR_PASSWORD` and `TURN_SHARED_SECRET_FROM_NAUTILUS` with whatever password you'd like and the turn-secret from Nautilus Support respectively
- Do **not** include the square brackets when you type your command out

This is an idea of how your secret is formatted.
```
{
    my-pass: 
        my-password: 
            YOUR_PASSWORD: ****
        turn-secret:
            TURN_SHARED_SECRET_FROM_NAUTILUS: **********************
}
```

Of the 5 things in square brackets, you are **required** to change 3 things: `my-pass`, `YOUR_PASSWORD`, and `TURN_SHARED_SECRET_FROM_NAUTILUS`. It is recommended you change `my-password` and `turn-secret` as well, but not required.  

## 5. Editing the Pod GUI File

The other yaml file in this repository, `xgl.yaml`, is the actual Pod that will be running the GUI desktop. When you are ready, you can try running a Deployment version of it (you'll need to a few more options not covered in this guide).

To get the Pod working you'll first need to make a couple changes to the file:

- Change the name of your Pod on line 4
- Change the value of the secret name on line 29 (your unique name that you replaced `my-pass` with)
- Change the value of the key to your password **reference** on line 30 (if you did not change `my-password` when creating your Secret, put `my-password` here, otherwise change it accordingly)
- Change the value of claimName to your cache PVC name on line 91
- Change the value of claimName to your storage PVC name on line 94

## 6. Run the Pod
`kubectl create -f [your-xgl-filename].yaml`

## 7. Check for Errors
Check pod status using `kubectl get pods`
If you face any issues that occur regarding your pod, check and debug using this command `kubectl describe pod [pod-name]`

## 8. Port-forward the Pod
Run - `kubectl port-forward pod/[pod-name] 8080:8080`
<br />You should see an output that looks like this: `Forwarding from 127.0.0.1:8080 -> 8080`
<br />Copy and paste `127.0.0.1:8080` into your web browser 

*Make sure to run this command in your local terminal where you have access to a web browser*

## 9. Localhost page 
The page should prompt you to enter the username which is "ubuntu" and the password you set (`YOUR_PASSWORD`) in your Nautilus Secret.

## 10. You're Done!

Note that this dockerimage does not have the latest version of a lot of things built into the terminal, so you may need to run a couple commands like `sudo apt update`.
