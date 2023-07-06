# Nuke built-in rules and variables.
override MAKEFLAGS += -rR

override IMAGE_NAME := barebones

.PHONY: all
all: $(IMAGE_NAME).hdd

.PHONY: run-uefi
run-hdd-uefi: ovmf $(IMAGE_NAME).hdd
	qemu-system-riscv64 \
        -m 512M \
        -M virt \
        -cpu rv64 \
        -drive if=pflash,unit=1,format=raw,file=ovmf/OVMF.fd \
        -net none \
        -smp 4 \
        -device ramfb \
        -device qemu-xhci \
        -device usb-kbd \
        -device virtio-blk-device,drive=hd0 \
        -drive id=hd0,format=raw,file=${IMAGE_NAME}.hdd \
        -serial stdio

ovmf:
	mkdir -p ovmf
	cd ovmf && curl -o OVMF.fd https://retrage.github.io/edk2-nightly/bin/RELEASERISCV64_VIRT.fd && dd if=/dev/zero of=OVMF.fd bs=1 count=0 seek=33554432

limine:
	git clone https://github.com/limine-bootloader/limine.git --branch=v5.x-branch-binary --depth=1
	$(MAKE) -C limine

.PHONY: kernel
kernel:
	$(MAKE) -C kernel

$(IMAGE_NAME).hdd: limine kernel
	rm -f $(IMAGE_NAME).hdd
	dd if=/dev/zero bs=1M count=0 seek=64 of=$(IMAGE_NAME).hdd
	parted -s $(IMAGE_NAME).hdd mklabel gpt
	parted -s $(IMAGE_NAME).hdd mkpart ESP fat32 2048s 100%
	parted -s $(IMAGE_NAME).hdd set 1 esp on
	./limine/limine bios-install $(IMAGE_NAME).hdd
	sudo losetup -Pf --show $(IMAGE_NAME).hdd >loopback_dev
	sudo mkfs.fat -F 32 `cat loopback_dev`p1
	mkdir -p img_mount
	sudo mount `cat loopback_dev`p1 img_mount
	sudo mkdir -p img_mount/EFI/BOOT
	sudo cp -v kernel/kernel.elf limine.cfg limine/limine-bios.sys img_mount/
	sudo cp -v limine/BOOT*.EFI img_mount/EFI/BOOT/
	sync
	sudo umount img_mount
	sudo losetup -d `cat loopback_dev`
	rm -rf loopback_dev img_mount

.PHONY: clean
clean:
	rm -rf $(IMAGE_NAME).hdd
	$(MAKE) -C kernel clean

.PHONY: distclean
distclean: clean
	rm -rf limine ovmf
	$(MAKE) -C kernel distclean
