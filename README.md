# আমার নিজের ৬৪-বিট অপারেটিং সিস্টেম কার্নেল

এটা একটা bare-metal x86_64 OS কার্নেলের শুরুর কাঠামো (multiboot2 + GRUB দিয়ে বুট হয়)।
স্ক্রিনে শুধু `OK` লেখা প্রিন্ট করে — এটাই প্রথম মাইলফলক।

## ১. দরকারি টুলস
- টেক্সট এডিটর (VS Code)
- Docker (বিল্ড এনভায়রনমেন্টের জন্য)
- QEMU (OS রান/emulate করার জন্য) — PATH-এ যুক্ত থাকতে হবে

## ২. বিল্ড এনভায়রনমেন্ট তৈরি
```
docker build buildenv -t myos-buildenv
```

## ৩. বিল্ড এনভায়রনমেন্টে প্রবেশ
Linux/macOS/WSL/git-bash:
```
docker run --rm -it -v "$(pwd)":/root/env myos-buildenv
```
Windows CMD:
```
docker run --rm -it -v "%cd%":/root/env myos-buildenv
```
Windows PowerShell:
```
docker run --rm -it -v "${pwd}:/root/env" myos-buildenv
```

## ৪. কার্নেল বিল্ড করা (কন্টেইনারের ভিতরে)
```
make build-x86_64
```
সফল হলে `dist/x86_64/kernel.iso` তৈরি হবে।
কন্টেইনার থেকে বের হতে: `exit`

## ৫. QEMU দিয়ে রান করা
```
qemu-system-x86_64 -cdrom dist/x86_64/kernel.iso
```
স্ক্রিনে সাদা ব্যাকগ্রাউন্ডে সবুজ `OK` দেখা গেলে সব ঠিক আছে!

BIOS এরর হলে:
- Windows: `qemu-system-x86_64 -cdrom dist/x86_64/kernel.iso -L "C:\Program Files\qemu"`
- Linux: `qemu-system-x86_64 -cdrom dist/x86_64/kernel.iso -L /usr/share/qemu/`

## ৬. পরিষ্কার করা (Cleanup)
```
docker rmi myos-buildenv -f
```

## ফাইল স্ট্রাকচার ও কাজ
- `buildenv/Dockerfile` — বিল্ড টুলচেইন (cross-compiler, nasm, grub, xorriso)
- `src/impl/x86_64/boot/header.asm` — Multiboot2 হেডার (GRUB এটা খুঁজে বুট করে)
- `src/impl/x86_64/boot/main.asm` — কার্নেলের এন্ট্রি পয়েন্ট, VGA মেমোরিতে `OK` লেখে
- `targets/x86_64/linker.ld` — লিংকার স্ক্রিপ্ট, মেমোরি লেআউট ঠিক করে (1MB থেকে শুরু)
- `targets/x86_64/iso/boot/grub/grub.cfg` — GRUB মেনু কনফিগ
- `Makefile` — পুরো বিল্ড প্রসেস অটোমেট করে (asm → .o → kernel.bin → kernel.iso)
