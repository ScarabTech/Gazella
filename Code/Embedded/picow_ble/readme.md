with raspberry pi pico vs code extension installed select compile on the bottom ribbon

mount the mass storage device manually:
$ dmesg | tail
[ 371.973555] sd 0:0:0:0: [sda] Attached SCSI removable disk
$ sudo mkdir -p /mnt/pico
$ sudo mount /dev/sda1 /mnt/pico
If you can see files in /mnt/pico, the USB Mass Storage Device has mounted correctly:
$ ls /mnt/pico/
INDEX.HTM INFO_UF2.TXT
Copy your blink.uf2 onto the device:
$ sudo cp blink.uf2 /mnt/pico
$ sudo sync
The microcontroller automatically disconnects as a USB Mass Storage Device and runs your code, but just to be safe,
you should unmount manually as well:
$ sudo umount /mnt/pico

Environment variables (add these to .bashrc):
export PICO_SDK_PATH=/home/pi/pico/pico-sdk
export PICO_EXAMPLES_PATH=/home/pi/pico/pico-examples
export PICO_EXTRAS_PATH=/home/pi/pico/pico-extras
export PICO_PLAYGROUND_PATH=/home/pi/pico/pico-playground

Building examples (section 3.1, page 8):
cd pico-examples
mkdir build
cd build
cmake ..
cd blink
make -j4

Running over USB by drag & drop (section 3.2.1, page 10):
Connect the Raspberry Pi Pico to your Raspberry Pi using a micro-USB cable,
making sure that you hold down the BOOTSEL button to force it into USB Mass Storage Mode.
The Raspberry Pi Pico should automatically mount as a USB Mass Storage Device.
From here, you can Drag-and-drop blink.uf2 onto the Mass Storage Device.

Running over USB by command line (section 3.2.2, page 10):
dmesg | tail
output is: [ 371.973555] sd 0:0:0:0: [sda] Attached SCSI removable disk
sudo mkdir -p /mnt/pico
sudo mount /dev/sda1 /mnt/pico
ls /mnt/pico/
output is: INDEX.HTM INFO_UF2.TXT
sudo cp blink.uf2 /mnt/pico
sudo sync
sudo umount /mnt/pico

Running over SWD (section 5.3, page 20):
Run the following command to program the resulting .elf file over SWD, and run it:
openocd -f interface/raspberrypi-swd.cfg -f target/rp2040.cfg -c "program blink/blink.elf verify reset exit"

Seeing output from pico
Serial input (stdin) and output (stdout) can be directed to either serial UART or to USB CDC (USB serial).
However by default stdio and printf will target the default Raspberry Pi Pico UART0.

Set output in CMakeLists.txt:
# enable usb output, disable uart output
pico_enable_stdio_usb(hello_usb 1)
pico_enable_stdio_uart(hello_usb 0)

Seeing output from pico USB (section 4.4, page 14):
minicom -b 115200 -o -D /dev/ttyACM0
-OR-
screen /dev/ttyACM0 115200

Seeing output from pico serial, UART0 (section 4.5, page 15):
Enable serial comm on the host (R PI 3B+):
sudo raspi-config
Add serial initialization to the pico program:
stdio_init_all();
uart_init(UART_ID, BAUD_RATE); // Set up our UART
gpio_set_function(UART_TX_PIN, GPIO_FUNC_UART);
gpio_set_function(UART_RX_PIN, GPIO_FUNC_UART);
Run a terminal program on the host:
minicom -b 115200 -o -D /dev/serial0
-OR-
screen /dev/serial0 115200

Debugging (section 6.3, page 22):
On host, run on one terminal:
openocd -f interface/raspberrypi-swd.cfg -f target/rp2040.cfg

on another terminal:
cd ~/pico/pico-examples/build/hello_world/serial
gdb-multiarch hello_serial.elf
(gdb) target remote localhost:3333
(gdb) load
(gdb) monitor reset init
(gdb) continue
(gdb) b main
(gdb) continue

When using SWD for debugging you need to use a UART based serial
connection (see Chapter 4) as the USB stack will be paused when the
RP2040 cores are stopped during debugging, which will cause any
attached USB devices to disconnect. You cannot use a USB CDC serial
connection during debugging.

Creating a project (section 8, page 30):
cd /home/pi/pico
mkdir test
cd test
create a test.c file
create a CMakeLists.txt file
cp ../pico-sdk/external/pico_sdk_import.cmake .
mkdir build
cd build
export PICO_SDK_PATH=../../pico-sdk
cmake ..
make -j4

Debugging your project (section 8.1, page 32):
cd ~/pico/test
rmdir build
mkdir build
cd build
export PICO_SDK_PATH=../../pico-sdk
cmake -DCMAKE_BUILD_TYPE=Debug ..
make
openocd -f interface/raspberrypi-swd.cfg -f target/rp2040.cfg

on another terminal:
cd ~/pico/test/build
gdb-multiarch test.elf
(gdb) target remote localhost:3333
(gdb) load
(gdb) monitor reset init // SSC: run this again if it bombs.
(gdb) continue

Automated Project generation (section 8.3, page 33):
cd /home/pi/pico/pico-project-generator
./pico_project.py --gui

Appendix E: Building the SDK API
documentation
$ cd pico-sdk
$ mkdir build
$ cd build
$ cmake -DPICO_EXAMPLES_PATH=../../pico-examples ..
$ make docs