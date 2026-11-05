cmake_minimum_required(VERSION 3.10)
project(
    draft-ietf-dtn-adm-yang
    LANGUAGES
)

# Inherit variables from top-level project
set(DRAFT_NAME "${PROJECT_NAME}")
set(DOC_INCLUDES
    "example-adm.yang"
    "modules/ietf-amm.yang"
    "modules/ietf-amm-base.yang"
    "modules/ietf-amm-semtype.yang"
    "modules/ietf-network-base.yang"
    "modules/iana-display-hints.yang"
    "modules/ietf-dtnma-agent.yang"
    "modules/ietf-dtnma-agent-acl.yang"
)

find_program(XML2RFC xml2rfc)
if(NOT XML2RFC)
    message(FATAL_ERROR "Program xml2rfc not found")
endif()
find_program(XMLSTARLET xmlstarlet)
if(NOT XMLSTARLET)
    message(FATAL_ERROR "Program xmlstarlet not found")
endif()
find_program(ASPELL aspell)
if(NOT ASPELL)
    message(FATAL_ERROR "Program aspell not found")
endif()
find_program(PYTHON python3)
if(NOT PYTHON)
    message(FATAL_ERROR "Program python3 not found")
endif()
set(SPLIT_ARTWORK "${CMAKE_CURRENT_SOURCE_DIR}/split_artwork.py")

set(DRAFT_NAME_XML "${CMAKE_CURRENT_SOURCE_DIR}/${DRAFT_NAME}.xml")
set(DRAFT_NAME_TXT "${DRAFT_NAME}.txt")
set(DRAFT_NAME_HTML "${DRAFT_NAME}.html")
set(DRAFT_NAME_PDF "${DRAFT_NAME}.pdf")
set(DRAFT_NAME_EXPAND "${DRAFT_NAME}-exp.xml")
add_custom_command(
    OUTPUT "${DRAFT_NAME_TXT}"
    DEPENDS "${DRAFT_NAME_XML}" ${DOC_INCLUDES}
    COMMAND ${XML2RFC} --no-network --allow-local-file-access --text -o "${DRAFT_NAME_TXT}" "${DRAFT_NAME_XML}"
)
add_custom_command(
    OUTPUT "${DRAFT_NAME_HTML}"
    DEPENDS "${DRAFT_NAME_XML}" ${DOC_INCLUDES}
    COMMAND ${XML2RFC} --no-network --allow-local-file-access --html -o "${DRAFT_NAME_HTML}" "${DRAFT_NAME_XML}"
)
add_custom_command(
    OUTPUT "${DRAFT_NAME_PDF}"
    DEPENDS "${DRAFT_NAME_XML}" ${DOC_INCLUDES}
    COMMAND ${XML2RFC} --no-network --allow-local-file-access --pdf -o "${DRAFT_NAME_PDF}" "${DRAFT_NAME_XML}"
)
add_custom_command(
    OUTPUT "${DRAFT_NAME_EXPAND}"
    DEPENDS "${DRAFT_NAME_XML}" ${DOC_INCLUDES}
    COMMAND ${XML2RFC} --no-network --allow-local-file-access --expand -o "${DRAFT_NAME_EXPAND}" "${DRAFT_NAME_XML}"
)
add_custom_target(
    specs ALL
    DEPENDS 
        "${DRAFT_NAME_TXT}"
        "${DRAFT_NAME_HTML}"
        "${DRAFT_NAME_EXPAND}"
)
add_custom_target(
    pdf
    DEPENDS "${DRAFT_NAME_PDF}"
)

set(DICTIONARY_TXT "${CMAKE_CURRENT_SOURCE_DIR}/dictionary.txt")
set(SPELLCHECK_XSL "${CMAKE_CURRENT_SOURCE_DIR}/spellcheck.xsl")
set(MISSPELLING_TXT "misspelling.txt")
add_custom_command(
    OUTPUT "dictionary.cwl"
    DEPENDS "${DICTIONARY_TXT}"
    COMMAND cat "${DICTIONARY_TXT}" |
        ${ASPELL} --lang=en create master "./dictionary.cwl"
)
add_custom_command(
    OUTPUT "${MISSPELLING_TXT}"
    DEPENDS "${DRAFT_NAME_XML}" "${SPELLCHECK_XSL}" "dictionary.cwl"
    COMMAND ${XMLSTARLET} tr "${SPELLCHECK_XSL}" "${DRAFT_NAME_XML}" |
        ${ASPELL} --mode=sgml --lang=EN_US --extra-dicts=./dictionary.cwl list |
        sort | uniq > "${MISSPELLING_TXT}"
)
add_custom_target(
    misspelling ALL
    DEPENDS "${MISSPELLING_TXT}"
    COMMAND cat "${MISSPELLING_TXT}"
    # success means file is present and empty
    COMMAND test -f "${MISSPELLING_TXT}" -a ! -s "${MISSPELLING_TXT}"
    COMMENT "Checking ${MISSPELLING_TXT}"
)

install(
    FILES
        "${CMAKE_CURRENT_BINARY_DIR}/${DRAFT_NAME_TXT}"
        "${CMAKE_CURRENT_BINARY_DIR}/${DRAFT_NAME_HTML}"
        "${CMAKE_CURRENT_BINARY_DIR}/${DRAFT_NAME_EXPAND}"
        "${CMAKE_CURRENT_BINARY_DIR}/${MISSPELLING_TXT}"
    TYPE DOC
)
