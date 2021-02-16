## Kernel

Posiblemente de problemas del kernel...

    sudo pacman -S linux-headers-$(uname -r)
 
 Could not open /dev/vmmon: No existe el fichero o el directorio. Please make sure that the kernel module `vmmon’ is loaded.
 
    sudo /etc/init.d/vmware start

Tambien:

    sudo modprobe -a vmw_vmci vmmon
    
   
