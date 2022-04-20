#### Install
# The only target in this package is the install target. fhelp files must be installed
# into the right location at install time
install(
  DIRECTORY fhelp_files/
  DESTINATION ${FERMI_INSTALL_HELPDIR}
)
