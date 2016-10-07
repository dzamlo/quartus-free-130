# Maintainer: David Manouchehri <manouchehri@riseup.net>
# Contributor: Simon Hollingshead <me at [firstnamelastname] dot com>
# Contributor: Matthias Blaicher <matthias at blaicher dot com>
# Contributor: cyrozap <cyrozap@gmail.com>
#
# NOTE: If you plan on using the usbblaster make sure you are member of the plugdev group.
# NOTE: Altera has dramatically changed their packing in regards to version 12. This
#       PKGBUILD will install the full Altera suite now. Be aware that the space requirement
#       is around 15GB now.
#
_basename=quartus-free
pkgname="${_basename}-130"
pkgver=13.0.1.232
pkgrel=2
pkgdesc="Quartus II 13.0 Web Edition, the last version to support Cyclone II and earlier FPGAs."
arch=('i686' 'x86_64')
url="http://www.altera.com/products/software/quartus-ii/web-edition"
license=('custom')
provides=("${_basename}")
conflicts=("${_basename}")

_alteradir="/opt/altera"

# Change this to "ae" if using ModelSim Subscription Edition
_modelsimver="ase"

# According to the installer script, these dependencies are needed for the installer
if [[ $CARCH = i686 ]]
then
   depends=('desktop-file-utils' 'expat' 'fontconfig' 'freetype2' 
   'glibc' 'gtk2' 'libcanberra' 'libpng' 'libpng12' 'libice' 'libsm' 
   'util-linux' 'ncurses' 'ncurses5-compat-libs' 'tcl' 'tcllib'
   'zlib' 'libx11' 'libxau' 'libxdmcp' 'libxext' 'libxft'
   'libxrender' 'libxt' 'libxtst')
fi

if [[ $CARCH = x86_64 ]]
then
   depends=('desktop-file-utils' 'expat' 'fontconfig' 'freetype2' 
   'glibc' 'gtk2' 'libcanberra' 'libpng' 'libpng12' 'libice' 'libsm' 
   'util-linux' 'ncurses' 'tcl' 'tcllib' 'zlib' 'libx11' 'libxau' 
   'libxdmcp' 'libxext' 'libxft' 'libxrender' 'libxt' 'libxtst'
   
   'lib32-expat' 'lib32-fontconfig' 'lib32-freetype2' 'lib32-glibc' 
   'lib32-gtk2' 'lib32-libcanberra' 'lib32-libpng' 'lib32-libpng12' 
   'lib32-libice' 'lib32-libsm' 'lib32-util-linux' 'lib32-ncurses' 
   'lib32-ncurses5-compat-libs' 'lib32-zlib' 'lib32-libx11' 
   'lib32-libxau' 'lib32-libxdmcp' 'lib32-libxext' 'lib32-libxft'
   'lib32-libxrender' 'lib32-libxt' 'lib32-libxtst')
fi

makedepends=('bash')
				  
source=("Quartus-web-13.0.1.232-linux.tar::http://download.altera.com/akdlm/software/acdsinst/13.0sp1/232/ib_tar/Quartus-web-13.0.1.232-linux.tar"
        "https://archive.archlinux.org/packages/l/lib32-freetype2/lib32-freetype2-2.5.0.1-1-x86_64.pkg.tar.xz"
	"quartus.sh" "quartus.desktop" "51-usbblaster.rules" "modelsim-ase.desktop" "modelsim-ae.desktop")
md5sums=('7588ed734761f62ec8f86b07a5adfffd'
         'd3b3b7cdf874b6dd0b60c40d84dd9128'
         '067c444cae7fe31d3608245712b43ce8'
         '32b17cb8b992fc2dccd33d87f0dcd8ce'
         'e1d0799c3c5558b9444a5b539254be84'
         '671f0f8b52afbf2dae842d91855fd90d'
         'd32c0fe838c76c77b4c721a621c221f7')

options=('!strip' '!upx') # Stripping and UPX will takes ages, I'd avoid it.

PKGEXT=".pkg.tar" # Same as above, compressing takes too long.

# Need to patch some Quartus/ModelSim files 
prepare() {
    cd "${srcdir}"
    
    echo "Notice: May require up to 54GB of free space during package building!"
    echo "Extracting install binaries and scripts from downloaded tar..."
    # Run setup.sh to extract Quartus, ModelSim, and device/help files
    DISPLAY="" bash ./setup.sh --mode unattended --unattendedmodeui none --installdir "${srcdir}${_alteradir}"
    echo "Finished extracting binaries and scripts."

    # Remove uninstaller and install logs since we have a working package management
    rm -r "${srcdir}${_alteradir}/uninstall"
    rm -r "${srcdir}${_alteradir}/logs"
    
    # Replace altera directory in integration files
    sed -i.bak "s,_alteradir,$_alteradir,g" quartus.sh
    sed -i.bak "s,_alteradir,$_alteradir,g" quartus.desktop
    sed -i.bak "s,_alteradir,$_alteradir,g" "modelsim-${_modelsimver}.desktop"
    
    # Fix modelsim startup code for Linux Kernel >=4.0
    # see https://wiki.archlinux.org/index.php/Altera_Design_Software
    sed -i.bak "s,linux_rh60,linux,g" "${srcdir}${_alteradir}/modelsim_${_modelsimver}/vco"  

    # Tell ModelSim where to find the specific lib32-freetype2 libraries it requires
    # see https://wiki.archlinux.org/index.php/Altera_Design_Software#With_freetype2_2.5.0.1-1
    sed -i.bak2 "10s,^,export LD_LIBRARY_PATH=${_alteradir}/modelsim_${_modelsimver}/lib \n,g" "${srcdir}${_alteradir}/modelsim_${_modelsimver}/vco"
}

package() {
    # Copy Quartus over to pkgdir for installation
    mkdir -p "${pkgdir}${_alteradir}"
    cp -r "${srcdir}${_alteradir}"/* "${pkgdir}${_alteradir}"

    # Place the lib32-freetype2 libraries where ModelSim is now expecting them to be
    # see https://wiki.archlinux.org/index.php/Altera_Design_Software#With_freetype2_2.5.0.1-1
    mkdir -p "${pkgdir}${_alteradir}/modelsim_${_modelsimver}/lib"
    install -D -m755 usr/lib32/libfreetype.so* "${pkgdir}${_alteradir}/modelsim_${_modelsimver}/lib"
    
    # Copy license file
    install -D -m644 "${pkgdir}${_alteradir}/quartus/license.txt" "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"


    # Install integration files
    install -D -m755 quartus.sh "${pkgdir}/etc/profile.d/quartus.sh"
    install -D -m644 51-usbblaster.rules "${pkgdir}/etc/udev/rules.d/51-usbblaster.rules"
    install -D -m644 quartus.desktop "${pkgdir}/usr/share/applications/quartus.desktop"
    install -D -m644 modelsim-${_modelsimver}.desktop "${pkgdir}/usr/share/applications/modelsim-${_modelsimver}.desktop"
}
