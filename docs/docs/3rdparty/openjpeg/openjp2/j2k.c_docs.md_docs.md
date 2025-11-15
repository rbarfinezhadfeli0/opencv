# Documentation for `docs/3rdparty/openjpeg/openjp2/j2k.c_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/openjpeg/openjp2/j2k.c_docs.md`
- **File Name**: `j2k.c_docs.md`
- **File Size**: 104,625 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/openjpeg/openjp2/j2k.c_docs.md](../../../../docs/3rdparty/openjpeg/openjp2/j2k.c_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/openjpeg/openjp2` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/openjpeg/openjp2/j2k.c`

## File Metadata

- **Full Path**: `3rdparty/openjpeg/openjp2/j2k.c`
- **File Name**: `j2k.c`
- **File Size**: 514,588 bytes
- **File Type**: .c
- **Link to Source**: [3rdparty/openjpeg/openjp2/j2k.c](../../../3rdparty/openjpeg/openjp2/j2k.c)

## Purpose and Role

This file is located in the `3rdparty/openjpeg/openjp2` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*
 * The copyright in this software is being made available under the 2-clauses
 * BSD License, included below. This software may be subject to other third
 * party and contributor rights, including patent rights, and no such rights
 * are granted under this license.
 *
 * Copyright (c) 2002-2014, Universite catholique de Louvain (UCL), Belgium
 * Copyright (c) 2002-2014, Professor Benoit Macq
 * Copyright (c) 2001-2003, David Janssens
 * Copyright (c) 2002-2003, Yannick Verschueren
 * Copyright (c) 2003-2007, Francois-Olivier Devaux
 * Copyright (c) 2003-2014, Antonin Descampe
 * Copyright (c) 2005, Herve Drolon, FreeImage Team
 * Copyright (c) 2008, Jerome Fimes, Communications & Systemes <jerome.fimes@c-s.fr>
 * Copyright (c) 2006-2007, Parvatha Elangovan
 * Copyright (c) 2010-2011, Kaori Hagihara
 * Copyright (c) 2011-2012, Centre National d'Etudes Spatiales (CNES), France
 * Copyright (c) 2012, CS Systemes d'Information, France
 * Copyright (c) 2017, IntoPIX SA <support@intopix.com>
 * All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions
 * are met:
 * 1. Redistributions of source code must retain the above copyright
 *    notice, this list of conditions and the following disclaimer.
 * 2. Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in the
 *    documentation and/or other materials provided with the distribution.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS `AS IS'
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
 * ARE DISCLAIMED.  IN NO EVENT SHALL THE COPYRIGHT OWNER OR CONTRIBUTORS BE
 * LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 * CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
 * SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
 * INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
 * CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
 * ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
 * POSSIBILITY OF SUCH DAMAGE.
 */

#include "opj_includes.h"

/** @defgroup J2K J2K - JPEG-2000 codestream reader/writer */
/*@{*/

/** @name Local static functions */
/*@{*/

/**
 * Sets up the procedures to do on reading header. Developers wanting to extend the library can add their own reading procedures.
 */
static OPJ_BOOL opj_j2k_setup_header_reading(opj_j2k_t *p_j2k,
        opj_event_mgr_t * p_manager);

/**
 * The read header procedure.
 */
static OPJ_BOOL opj_j2k_read_header_procedure(opj_j2k_t *p_j2k,
        opj_stream_private_t *p_stream,
        opj_event_mgr_t * p_manager);

/**
 * The default encoding validation procedure without any extension.
 *
 * @param       p_j2k                   the jpeg2000 codec to validate.
 * @param       p_stream                the input stream to validate.
 * @param       p_manager               the user event manager.
 *
 * @return true if the parameters are correct.
 */
static OPJ_BOOL opj_j2k_encoding_validation(opj_j2k_t * p_j2k,
        opj_stream_private_t *p_stream,
        opj_event_mgr_t * p_manager);

/**
 * The default decoding validation procedure without any extension.
 *
 * @param       p_j2k                   the jpeg2000 codec to validate.
 * @param       p_stream                                the input stream to validate.
 * @param       p_manager               the user event manager.
 *
 * @return true if the parameters are correct.
 */
static OPJ_BOOL opj_j2k_decoding_validation(opj_j2k_t * p_j2k,
        opj_stream_private_t *p_stream,
        opj_event_mgr_t * p_manager);

/**
 * Sets up the validation ,i.e. adds the procedures to launch to make sure the codec parameters
 * are valid. Developers wanting to extend the library can add their own validation procedures.
 */
static OPJ_BOOL opj_j2k_setup_encoding_validation(opj_j2k_t *p_j2k,
        opj_event_mgr_t * p_manager);

/**
 * Sets up the validation ,i.e. adds the procedures to launch to make sure the codec parameters
 * are valid. Developers wanting to extend the library can add their own validation procedures.
 */
static OPJ_BOOL opj_j2k_setup_decoding_validation(opj_j2k_t *p_j2k,
        opj_event_mgr_t * p_manager);

/**
 * Sets up the validation ,i.e. adds the procedures to launch to make sure the codec parameters
 * are valid. Developers wanting to extend the library can add their own validation procedures.
 */
static OPJ_BOOL opj_j2k_setup_end_compress(opj_j2k_t *p_j2k,
        opj_event_mgr_t * p_manager);

/**
 * The mct encoding validation procedure.
 *
 * @param       p_j2k                   the jpeg2000 codec to validate.
 * @param       p_stream                                the input stream to validate.
 * @param       p_manager               the user event manager.
 *
 * @return true if the parameters are correct.
 */
static OPJ_BOOL opj_j2k_mct_validation(opj_j2k_t * p_j2k,
                                       opj_stream_private_t *p_stream,
                                       opj_event_mgr_t * p_manager);

/**
 * Builds the tcd decoder to use to decode tile.
 */
static OPJ_BOOL opj_j2k_build_decoder(opj_j2k_t * p_j2k,
                                      opj_stream_private_t *p_stream,
                                      opj_event_mgr_t * p_manager);
/**
 * Builds the tcd encoder to use to encode tile.
 */
static OPJ_BOOL opj_j2k_build_encoder(opj_j2k_t * p_j2k,
                                      opj_stream_private_t *p_stream,
                                      opj_event_mgr_t * p_manager);

/**
 * Creates a tile-coder encoder.
 *
 * @param       p_stream                        the stream to write data to.
 * @param       p_j2k                           J2K codec.
 * @param       p_manager                   the user event manager.
*/
static OPJ_BOOL opj_j2k_create_tcd(opj_j2k_t *p_j2k,
                                   opj_stream_private_t *p_stream,
                                   opj_event_mgr_t * p_manager);

/**
 * Executes the given procedures on the given codec.
 *
 * @param       p_procedure_list        the list of procedures to execute
 * @param       p_j2k                           the jpeg2000 codec to execute the procedures on.
 * @param       p_stream                        the stream to execute the procedures on.
 * @param       p_manager                       the user manager.
 *
 * @return      true                            if all the procedures were successfully executed.
 */
static OPJ_BOOL opj_j2k_exec(opj_j2k_t * p_j2k,
                             opj_procedure_list_t * p_procedure_list,
                             opj_stream_private_t *p_stream,
                             opj_event_mgr_t * p_manager);

/**
 * Updates the rates of the tcp.
 *
 * @param       p_stream                                the stream to write data to.
 * @param       p_j2k                           J2K codec.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_update_rates(opj_j2k_t *p_j2k,
                                     opj_stream_private_t *p_stream,
                                     opj_event_mgr_t * p_manager);

/**
 * Copies the decoding tile parameters onto all the tile parameters.
 * Creates also the tile decoder.
 */
static OPJ_BOOL opj_j2k_copy_default_tcp_and_create_tcd(opj_j2k_t * p_j2k,
        opj_stream_private_t *p_stream,
        opj_event_mgr_t * p_manager);

/**
 * Destroys the memory associated with the decoding of headers.
 */
static OPJ_BOOL opj_j2k_destroy_header_memory(opj_j2k_t * p_j2k,
        opj_stream_private_t *p_stream,
        opj_event_mgr_t * p_manager);

/**
 * Reads the lookup table containing all the marker, status and action, and returns the handler associated
 * with the marker value.
 * @param       p_id            Marker value to look up
 *
 * @return      the handler associated with the id.
*/
static const struct opj_dec_memory_marker_handler * opj_j2k_get_marker_handler(
    OPJ_UINT32 p_id);

/**
 * Destroys a tile coding parameter structure.
 *
 * @param       p_tcp           the tile coding parameter to destroy.
 */
static void opj_j2k_tcp_destroy(opj_tcp_t *p_tcp);

/**
 * Destroys the data inside a tile coding parameter structure.
 *
 * @param       p_tcp           the tile coding parameter which contain data to destroy.
 */
static void opj_j2k_tcp_data_destroy(opj_tcp_t *p_tcp);

/**
 * Destroys a coding parameter structure.
 *
 * @param       p_cp            the coding parameter to destroy.
 */
static void opj_j2k_cp_destroy(opj_cp_t *p_cp);

/**
 * Compare 2 a SPCod/ SPCoc elements, i.e. the coding style of a given component of a tile.
 *
 * @param       p_j2k            J2K codec.
 * @param       p_tile_no        Tile number
 * @param       p_first_comp_no  The 1st component number to compare.
 * @param       p_second_comp_no The 1st component number to compare.
 *
 * @return OPJ_TRUE if SPCdod are equals.
 */
static OPJ_BOOL opj_j2k_compare_SPCod_SPCoc(opj_j2k_t *p_j2k,
        OPJ_UINT32 p_tile_no, OPJ_UINT32 p_first_comp_no, OPJ_UINT32 p_second_comp_no);

/**
 * Writes a SPCod or SPCoc element, i.e. the coding style of a given component of a tile.
 *
 * @param       p_j2k           J2K codec.
 * @param       p_tile_no       FIXME DOC
 * @param       p_comp_no       the component number to output.
 * @param       p_data          FIXME DOC
 * @param       p_header_size   FIXME DOC
 * @param       p_manager       the user event manager.
 *
 * @return FIXME DOC
*/
static OPJ_BOOL opj_j2k_write_SPCod_SPCoc(opj_j2k_t *p_j2k,
        OPJ_UINT32 p_tile_no,
        OPJ_UINT32 p_comp_no,
        OPJ_BYTE * p_data,
        OPJ_UINT32 * p_header_size,
        opj_event_mgr_t * p_manager);

/**
 * Gets the size taken by writing a SPCod or SPCoc for the given tile and component.
 *
 * @param       p_j2k                   the J2K codec.
 * @param       p_tile_no               the tile index.
 * @param       p_comp_no               the component being outputted.
 *
 * @return      the number of bytes taken by the SPCod element.
 */
static OPJ_UINT32 opj_j2k_get_SPCod_SPCoc_size(opj_j2k_t *p_j2k,
        OPJ_UINT32 p_tile_no,
        OPJ_UINT32 p_comp_no);

/**
 * Reads a SPCod or SPCoc element, i.e. the coding style of a given component of a tile.
 * @param       p_j2k           the jpeg2000 codec.
 * @param       compno          FIXME DOC
 * @param       p_header_data   the data contained in the COM box.
 * @param       p_header_size   the size of the data contained in the COM marker.
 * @param       p_manager       the user event manager.
*/
static OPJ_BOOL opj_j2k_read_SPCod_SPCoc(opj_j2k_t *p_j2k,
        OPJ_UINT32 compno,
        OPJ_BYTE * p_header_data,
        OPJ_UINT32 * p_header_size,
        opj_event_mgr_t * p_manager);

/**
 * Gets the size taken by writing SQcd or SQcc element, i.e. the quantization values of a band in the QCD or QCC.
 *
 * @param       p_tile_no               the tile index.
 * @param       p_comp_no               the component being outputted.
 * @param       p_j2k                   the J2K codec.
 *
 * @return      the number of bytes taken by the SPCod element.
 */
static OPJ_UINT32 opj_j2k_get_SQcd_SQcc_size(opj_j2k_t *p_j2k,
        OPJ_UINT32 p_tile_no,
        OPJ_UINT32 p_comp_no);

/**
 * Compares 2 SQcd or SQcc element, i.e. the quantization values of a band in the QCD or QCC.
 *
 * @param       p_j2k                   J2K codec.
 * @param       p_tile_no               the tile to output.
 * @param       p_first_comp_no         the first component number to compare.
 * @param       p_second_comp_no        the second component number to compare.
 *
 * @return OPJ_TRUE if equals.
 */
static OPJ_BOOL opj_j2k_compare_SQcd_SQcc(opj_j2k_t *p_j2k,
        OPJ_UINT32 p_tile_no, OPJ_UINT32 p_first_comp_no, OPJ_UINT32 p_second_comp_no);


/**
 * Writes a SQcd or SQcc element, i.e. the quantization values of a band in the QCD or QCC.
 *
 * @param       p_tile_no               the tile to output.
 * @param       p_comp_no               the component number to output.
 * @param       p_data                  the data buffer.
 * @param       p_header_size   pointer to the size of the data buffer, it is changed by the function.
 * @param       p_j2k                   J2K codec.
 * @param       p_manager               the user event manager.
 *
*/
static OPJ_BOOL opj_j2k_write_SQcd_SQcc(opj_j2k_t *p_j2k,
                                        OPJ_UINT32 p_tile_no,
                                        OPJ_UINT32 p_comp_no,
                                        OPJ_BYTE * p_data,
                                        OPJ_UINT32 * p_header_size,
                                        opj_event_mgr_t * p_manager);

/**
 * Updates the Tile Length Marker.
 */
static void opj_j2k_update_tlm(opj_j2k_t * p_j2k, OPJ_UINT32 p_tile_part_size);

/**
 * Reads a SQcd or SQcc element, i.e. the quantization values of a band in the QCD or QCC.
 *
 * @param       p_j2k           J2K codec.
 * @param       compno          the component number to output.
 * @param       p_header_data   the data buffer.
 * @param       p_header_size   pointer to the size of the data buffer, it is changed by the function.
 * @param       p_manager       the user event manager.
 *
*/
static OPJ_BOOL opj_j2k_read_SQcd_SQcc(opj_j2k_t *p_j2k,
                                       OPJ_UINT32 compno,
                                       OPJ_BYTE * p_header_data,
                                       OPJ_UINT32 * p_header_size,
                                       opj_event_mgr_t * p_manager);

/**
 * Copies the tile component parameters of all the component from the first tile component.
 *
 * @param               p_j2k           the J2k codec.
 */
static void opj_j2k_copy_tile_component_parameters(opj_j2k_t *p_j2k);

/**
 * Copies the tile quantization parameters of all the component from the first tile component.
 *
 * @param               p_j2k           the J2k codec.
 */
static void opj_j2k_copy_tile_quantization_parameters(opj_j2k_t *p_j2k);

/**
 * Reads the tiles.
 */
static OPJ_BOOL opj_j2k_decode_tiles(opj_j2k_t *p_j2k,
                                     opj_stream_private_t *p_stream,
                                     opj_event_mgr_t * p_manager);

static OPJ_BOOL opj_j2k_pre_write_tile(opj_j2k_t * p_j2k,
                                       OPJ_UINT32 p_tile_index,
                                       opj_stream_private_t *p_stream,
                                       opj_event_mgr_t * p_manager);

static OPJ_BOOL opj_j2k_update_image_data(opj_tcd_t * p_tcd,
        opj_image_t* p_output_image);

static void opj_get_tile_dimensions(opj_image_t * l_image,
                                    opj_tcd_tilecomp_t * l_tilec,
                                    opj_image_comp_t * l_img_comp,
                                    OPJ_UINT32* l_size_comp,
                                    OPJ_UINT32* l_width,
                                    OPJ_UINT32* l_height,
                                    OPJ_UINT32* l_offset_x,
                                    OPJ_UINT32* l_offset_y,
                                    OPJ_UINT32* l_image_width,
                                    OPJ_UINT32* l_stride,
                                    OPJ_UINT32* l_tile_offset);

static void opj_j2k_get_tile_data(opj_tcd_t * p_tcd, OPJ_BYTE * p_data);

static OPJ_BOOL opj_j2k_post_write_tile(opj_j2k_t * p_j2k,
                                        opj_stream_private_t *p_stream,
                                        opj_event_mgr_t * p_manager);

/**
 * Sets up the procedures to do on writing header.
 * Developers wanting to extend the library can add their own writing procedures.
 */
static OPJ_BOOL opj_j2k_setup_header_writing(opj_j2k_t *p_j2k,
        opj_event_mgr_t * p_manager);

static OPJ_BOOL opj_j2k_write_first_tile_part(opj_j2k_t *p_j2k,
        OPJ_BYTE * p_data,
        OPJ_UINT32 * p_data_written,
        OPJ_UINT32 total_data_size,
        opj_stream_private_t *p_stream,
        struct opj_event_mgr * p_manager);

static OPJ_BOOL opj_j2k_write_all_tile_parts(opj_j2k_t *p_j2k,
        OPJ_BYTE * p_data,
        OPJ_UINT32 * p_data_written,
        OPJ_UINT32 total_data_size,
        opj_stream_private_t *p_stream,
        struct opj_event_mgr * p_manager);

/**
 * Gets the offset of the header.
 *
 * @param       p_stream                the stream to write data to.
 * @param       p_j2k                   J2K codec.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_get_end_header(opj_j2k_t *p_j2k,
                                       opj_stream_private_t *p_stream,
                                       opj_event_mgr_t * p_manager);

static OPJ_BOOL opj_j2k_allocate_tile_element_cstr_index(opj_j2k_t *p_j2k);

/*
 * -----------------------------------------------------------------------
 * -----------------------------------------------------------------------
 * -----------------------------------------------------------------------
 */

/**
 * Writes the SOC marker (Start Of Codestream)
 *
 * @param       p_stream                        the stream to write data to.
 * @param       p_j2k                   J2K codec.
 * @param       p_manager       the user event manager.
*/
static OPJ_BOOL opj_j2k_write_soc(opj_j2k_t *p_j2k,
                                  opj_stream_private_t *p_stream,
                                  opj_event_mgr_t * p_manager);

/**
 * Reads a SOC marker (Start of Codestream)
 * @param       p_j2k           the jpeg2000 file codec.
 * @param       p_stream        XXX needs data
 * @param       p_manager       the user event manager.
*/
static OPJ_BOOL opj_j2k_read_soc(opj_j2k_t *p_j2k,
                                 opj_stream_private_t *p_stream,
                                 opj_event_mgr_t * p_manager);

/**
 * Writes the SIZ marker (image and tile size)
 *
 * @param       p_j2k           J2K codec.
 * @param       p_stream        the stream to write data to.
 * @param       p_manager       the user event manager.
*/
static OPJ_BOOL opj_j2k_write_siz(opj_j2k_t *p_j2k,
                                  opj_stream_private_t *p_stream,
                                  opj_event_mgr_t * p_manager);

/**
 * Reads a SIZ marker (image and tile size)
 * @param       p_j2k           the jpeg2000 file codec.
 * @param       p_header_data   the data contained in the SIZ box.
 * @param       p_header_size   the size of the data contained in the SIZ marker.
 * @param       p_manager       the user event manager.
*/
static OPJ_BOOL opj_j2k_read_siz(opj_j2k_t *p_j2k,
                                 OPJ_BYTE * p_header_data,
                                 OPJ_UINT32 p_header_size,
                                 opj_event_mgr_t * p_manager);

/**
 * Writes the COM marker (comment)
 *
 * @param       p_stream                        the stream to write data to.
 * @param       p_j2k                   J2K codec.
 * @param       p_manager       the user event manager.
*/
static OPJ_BOOL opj_j2k_write_com(opj_j2k_t *p_j2k,
                                  opj_stream_private_t *p_stream,
                                  opj_event_mgr_t * p_manager);

/**
 * Reads a COM marker (comments)
 * @param       p_j2k           the jpeg2000 file codec.
 * @param       p_header_data   the data contained in the COM box.
 * @param       p_header_size   the size of the data contained in the COM marker.
 * @param       p_manager       the user event manager.
*/
static OPJ_BOOL opj_j2k_read_com(opj_j2k_t *p_j2k,
                                 OPJ_BYTE * p_header_data,
                                 OPJ_UINT32 p_header_size,
                                 opj_event_mgr_t * p_manager);
/**
 * Writes the COD marker (Coding style default)
 *
 * @param       p_stream                        the stream to write data to.
 * @param       p_j2k                   J2K codec.
 * @param       p_manager       the user event manager.
*/
static OPJ_BOOL opj_j2k_write_cod(opj_j2k_t *p_j2k,
                                  opj_stream_private_t *p_stream,
                                  opj_event_mgr_t * p_manager);

/**
 * Reads a COD marker (Coding style defaults)
 * @param       p_header_data   the data contained in the COD box.
 * @param       p_j2k                   the jpeg2000 codec.
 * @param       p_header_size   the size of the data contained in the COD marker.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_read_cod(opj_j2k_t *p_j2k,
                                 OPJ_BYTE * p_header_data,
                                 OPJ_UINT32 p_header_size,
                                 opj_event_mgr_t * p_manager);

/**
 * Compares 2 COC markers (Coding style component)
 *
 * @param       p_j2k            J2K codec.
 * @param       p_first_comp_no  the index of the first component to compare.
 * @param       p_second_comp_no the index of the second component to compare.
 *
 * @return      OPJ_TRUE if equals
 */
static OPJ_BOOL opj_j2k_compare_coc(opj_j2k_t *p_j2k,
                                    OPJ_UINT32 p_first_comp_no, OPJ_UINT32 p_second_comp_no);

/**
 * Writes the COC marker (Coding style component)
 *
 * @param       p_j2k       J2K codec.
 * @param       p_comp_no   the index of the component to output.
 * @param       p_stream    the stream to write data to.
 * @param       p_manager   the user event manager.
*/
static OPJ_BOOL opj_j2k_write_coc(opj_j2k_t *p_j2k,
                                  OPJ_UINT32 p_comp_no,
                                  opj_stream_private_t *p_stream,
                                  opj_event_mgr_t * p_manager);

/**
 * Writes the COC marker (Coding style component)
 *
 * @param       p_j2k                   J2K codec.
 * @param       p_comp_no               the index of the component to output.
 * @param       p_data          FIXME DOC
 * @param       p_data_written  FIXME DOC
 * @param       p_manager               the user event manager.
*/
static void opj_j2k_write_coc_in_memory(opj_j2k_t *p_j2k,
                                        OPJ_UINT32 p_comp_no,
                                        OPJ_BYTE * p_data,
                                        OPJ_UINT32 * p_data_written,
                                        opj_event_mgr_t * p_manager);

/**
 * Gets the maximum size taken by a coc.
 *
 * @param       p_j2k   the jpeg2000 codec to use.
 */
static OPJ_UINT32 opj_j2k_get_max_coc_size(opj_j2k_t *p_j2k);

/**
 * Reads a COC marker (Coding Style Component)
 * @param       p_header_data   the data contained in the COC box.
 * @param       p_j2k                   the jpeg2000 codec.
 * @param       p_header_size   the size of the data contained in the COC marker.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_read_coc(opj_j2k_t *p_j2k,
                                 OPJ_BYTE * p_header_data,
                                 OPJ_UINT32 p_header_size,
                                 opj_event_mgr_t * p_manager);

/**
 * Writes the QCD marker (quantization default)
 *
 * @param       p_j2k                   J2K codec.
 * @param       p_stream                the stream to write data to.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_write_qcd(opj_j2k_t *p_j2k,
                                  opj_stream_private_t *p_stream,
                                  opj_event_mgr_t * p_manager);

/**
 * Reads a QCD marker (Quantization defaults)
 * @param       p_header_data   the data contained in the QCD box.
 * @param       p_j2k                   the jpeg2000 codec.
 * @param       p_header_size   the size of the data contained in the QCD marker.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_read_qcd(opj_j2k_t *p_j2k,
                                 OPJ_BYTE * p_header_data,
                                 OPJ_UINT32 p_header_size,
                                 opj_event_mgr_t * p_manager);

/**
 * Compare QCC markers (quantization component)
 *
 * @param       p_j2k                 J2K codec.
 * @param       p_first_comp_no       the index of the first component to compare.
 * @param       p_second_comp_no      the index of the second component to compare.
 *
 * @return OPJ_TRUE if equals.
 */
static OPJ_BOOL opj_j2k_compare_qcc(opj_j2k_t *p_j2k,
                                    OPJ_UINT32 p_first_comp_no, OPJ_UINT32 p_second_comp_no);

/**
 * Writes the QCC marker (quantization component)
 *
 * @param       p_comp_no       the index of the component to output.
 * @param       p_stream                the stream to write data to.
 * @param       p_j2k                   J2K codec.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_write_qcc(opj_j2k_t *p_j2k,
                                  OPJ_UINT32 p_comp_no,
                                  opj_stream_private_t *p_stream,
                                  opj_event_mgr_t * p_manager);

/**
 * Writes the QCC marker (quantization component)
 *
 * @param       p_j2k           J2K codec.
 * @param       p_comp_no       the index of the component to output.
 * @param       p_data          FIXME DOC
 * @param       p_data_written  the stream to write data to.
 * @param       p_manager       the user event manager.
*/
static void opj_j2k_write_qcc_in_memory(opj_j2k_t *p_j2k,
                                        OPJ_UINT32 p_comp_no,
                                        OPJ_BYTE * p_data,
                                        OPJ_UINT32 * p_data_written,
                                        opj_event_mgr_t * p_manager);

/**
 * Gets the maximum size taken by a qcc.
 */
static OPJ_UINT32 opj_j2k_get_max_qcc_size(opj_j2k_t *p_j2k);

/**
 * Reads a QCC marker (Quantization component)
 * @param       p_header_data   the data contained in the QCC box.
 * @param       p_j2k                   the jpeg2000 codec.
 * @param       p_header_size   the size of the data contained in the QCC marker.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_read_qcc(opj_j2k_t *p_j2k,
                                 OPJ_BYTE * p_header_data,
                                 OPJ_UINT32 p_header_size,
                                 opj_event_mgr_t * p_manager);
/**
 * Writes the POC marker (Progression Order Change)
 *
 * @param       p_stream                                the stream to write data to.
 * @param       p_j2k                           J2K codec.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_write_poc(opj_j2k_t *p_j2k,
                                  opj_stream_private_t *p_stream,
                                  opj_event_mgr_t * p_manager);
/**
 * Writes the POC marker (Progression Order Change)
 *
 * @param       p_j2k          J2K codec.
 * @param       p_data         FIXME DOC
 * @param       p_data_written the stream to write data to.
 * @param       p_manager      the user event manager.
 */
static void opj_j2k_write_poc_in_memory(opj_j2k_t *p_j2k,
                                        OPJ_BYTE * p_data,
                                        OPJ_UINT32 * p_data_written,
                                        opj_event_mgr_t * p_manager);
/**
 * Gets the maximum size taken by the writing of a POC.
 */
static OPJ_UINT32 opj_j2k_get_max_poc_size(opj_j2k_t *p_j2k);

/**
 * Reads a POC marker (Progression Order Change)
 *
 * @param       p_header_data   the data contained in the POC box.
 * @param       p_j2k                   the jpeg2000 codec.
 * @param       p_header_size   the size of the data contained in the POC marker.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_read_poc(opj_j2k_t *p_j2k,
                                 OPJ_BYTE * p_header_data,
                                 OPJ_UINT32 p_header_size,
                                 opj_event_mgr_t * p_manager);

/**
 * Gets the maximum size taken by the toc headers of all the tile parts of any given tile.
 */
static OPJ_UINT32 opj_j2k_get_max_toc_size(opj_j2k_t *p_j2k);

/**
 * Gets the maximum size taken by the headers of the SOT.
 *
 * @param       p_j2k   the jpeg2000 codec to use.
 */
static OPJ_UINT32 opj_j2k_get_specific_header_sizes(opj_j2k_t *p_j2k);

/**
 * Reads a CRG marker (Component registration)
 *
 * @param       p_header_data   the data contained in the TLM box.
 * @param       p_j2k                   the jpeg2000 codec.
 * @param       p_header_size   the size of the data contained in the TLM marker.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_read_crg(opj_j2k_t *p_j2k,
                                 OPJ_BYTE * p_header_data,
                                 OPJ_UINT32 p_header_size,
                                 opj_event_mgr_t * p_manager);
/**
 * Reads a TLM marker (Tile Length Marker)
 *
 * @param       p_header_data   the data contained in the TLM box.
 * @param       p_j2k                   the jpeg2000 codec.
 * @param       p_header_size   the size of the data contained in the TLM marker.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_read_tlm(opj_j2k_t *p_j2k,
                                 OPJ_BYTE * p_header_data,
                                 OPJ_UINT32 p_header_size,
                                 opj_event_mgr_t * p_manager);

/**
 * Writes the updated tlm.
 *
 * @param       p_stream                the stream to write data to.
 * @param       p_j2k                   J2K codec.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_write_updated_tlm(opj_j2k_t *p_j2k,
        opj_stream_private_t *p_stream,
        opj_event_mgr_t * p_manager);

/**
 * Reads a PLM marker (Packet length, main header marker)
 *
 * @param       p_header_data   the data contained in the TLM box.
 * @param       p_j2k                   the jpeg2000 codec.
 * @param       p_header_size   the size of the data contained in the TLM marker.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_read_plm(opj_j2k_t *p_j2k,
                                 OPJ_BYTE * p_header_data,
                                 OPJ_UINT32 p_header_size,
                                 opj_event_mgr_t * p_manager);
/**
 * Reads a PLT marker (Packet length, tile-part header)
 *
 * @param       p_header_data   the data contained in the PLT box.
 * @param       p_j2k                   the jpeg2000 codec.
 * @param       p_header_size   the size of the data contained in the PLT marker.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_read_plt(opj_j2k_t *p_j2k,
                                 OPJ_BYTE * p_header_data,
                                 OPJ_UINT32 p_header_size,
                                 opj_event_mgr_t * p_manager);

/**
 * Reads a PPM marker (Packed headers, main header)
 *
 * @param       p_header_data   the data contained in the POC box.
 * @param       p_j2k                   the jpeg2000 codec.
 * @param       p_header_size   the size of the data contained in the POC marker.
 * @param       p_manager               the user event manager.
 */

static OPJ_BOOL opj_j2k_read_ppm(
    opj_j2k_t *p_j2k,
    OPJ_BYTE * p_header_data,
    OPJ_UINT32 p_header_size,
    opj_event_mgr_t * p_manager);

/**
 * Merges all PPM markers read (Packed headers, main header)
 *
 * @param       p_cp      main coding parameters.
 * @param       p_manager the user event manager.
 */
static OPJ_BOOL opj_j2k_merge_ppm(opj_cp_t *p_cp, opj_event_mgr_t * p_manager);

/**
 * Reads a PPT marker (Packed packet headers, tile-part header)
 *
 * @param       p_header_data   the data contained in the PPT box.
 * @param       p_j2k                   the jpeg2000 codec.
 * @param       p_header_size   the size of the data contained in the PPT marker.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_read_ppt(opj_j2k_t *p_j2k,
                                 OPJ_BYTE * p_header_data,
                                 OPJ_UINT32 p_header_size,
                                 opj_event_mgr_t * p_manager);

/**
 * Merges all PPT markers read (Packed headers, tile-part header)
 *
 * @param       p_tcp   the tile.
 * @param       p_manager               the user event manager.
 */
static OPJ_BOOL opj_j2k_merge_ppt(opj_tcp_t *p_tcp,
                                  opj_event_mgr_t * p_manager);


/**
 * Writes the TLM marker (Tile Length Marker)
 *
 * @param       p_stream                                the stream to write data to.
 * @param       p_j2k                           J2K codec.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_write_tlm(opj_j2k_t *p_j2k,
                                  opj_stream_private_t *p_stream,
                                  opj_event_mgr_t * p_manager);

/**
 * Writes the SOT marker (Start of tile-part)
 *
 * @param       p_j2k            J2K codec.
 * @param       p_data           Output buffer
 * @param       total_data_size  Output buffer size
 * @param       p_data_written   Number of bytes written into stream
 * @param       p_stream         the stream to write data to.
 * @param       p_manager        the user event manager.
*/
static OPJ_BOOL opj_j2k_write_sot(opj_j2k_t *p_j2k,
                                  OPJ_BYTE * p_data,
                                  OPJ_UINT32 total_data_size,
                                  OPJ_UINT32 * p_data_written,
                                  const opj_stream_private_t *p_stream,
                                  opj_event_mgr_t * p_manager);

/**
 * Reads values from a SOT marker (Start of tile-part)
 *
 * the j2k decoder state is not affected. No side effects, no checks except for p_header_size.
 *
 * @param       p_header_data   the data contained in the SOT marker.
 * @param       p_header_size   the size of the data contained in the SOT marker.
 * @param       p_tile_no       Isot.
 * @param       p_tot_len       Psot.
 * @param       p_current_part  TPsot.
 * @param       p_num_parts     TNsot.
 * @param       p_manager       the user event manager.
 */
static OPJ_BOOL opj_j2k_get_sot_values(OPJ_BYTE *  p_header_data,
                                       OPJ_UINT32  p_header_size,
                                       OPJ_UINT32* p_tile_no,
                                       OPJ_UINT32* p_tot_len,
                                       OPJ_UINT32* p_current_part,
                                       OPJ_UINT32* p_num_parts,
                                       opj_event_mgr_t * p_manager);
/**
 * Reads a SOT marker (Start of tile-part)
 *
 * @param       p_header_data   the data contained in the SOT marker.
 * @param       p_j2k           the jpeg2000 codec.
 * @param       p_header_size   the size of the data contained in the PPT marker.
 * @param       p_manager       the user event manager.
*/
static OPJ_BOOL opj_j2k_read_sot(opj_j2k_t *p_j2k,
                                 OPJ_BYTE * p_header_data,
                                 OPJ_UINT32 p_header_size,
                                 opj_event_mgr_t * p_manager);
/**
 * Writes the SOD marker (Start of data)
 *
 * This also writes optional PLT markers (before SOD)
 *
 * @param       p_j2k               J2K codec.
 * @param       p_tile_coder        FIXME DOC
 * @param       p_data              FIXME DOC
 * @param       p_data_written      FIXME DOC
 * @param       total_data_size   FIXME DOC
 * @param       p_stream            the stream to write data to.
 * @param       p_manager           the user event manager.
*/
static OPJ_BOOL opj_j2k_write_sod(opj_j2k_t *p_j2k,
                                  opj_tcd_t * p_tile_coder,
                                  OPJ_BYTE * p_data,
                                  OPJ_UINT32 * p_data_written,
                                  OPJ_UINT32 total_data_size,
                                  const opj_stream_private_t *p_stream,
                                  opj_event_mgr_t * p_manager);

/**
 * Reads a SOD marker (Start Of Data)
 *
 * @param       p_j2k                   the jpeg2000 codec.
 * @param       p_stream                FIXME DOC
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_read_sod(opj_j2k_t *p_j2k,
                                 opj_stream_private_t *p_stream,
                                 opj_event_mgr_t * p_manager);

static void opj_j2k_update_tlm(opj_j2k_t * p_j2k, OPJ_UINT32 p_tile_part_size)
{
    if (p_j2k->m_specific_param.m_encoder.m_Ttlmi_is_byte) {
        opj_write_bytes(p_j2k->m_specific_param.m_encoder.m_tlm_sot_offsets_current,
                        p_j2k->m_current_tile_number, 1);
        p_j2k->m_specific_param.m_encoder.m_tlm_sot_offsets_current += 1;
    } else {
        opj_write_bytes(p_j2k->m_specific_param.m_encoder.m_tlm_sot_offsets_current,
                        p_j2k->m_current_tile_number, 2);
        p_j2k->m_specific_param.m_encoder.m_tlm_sot_offsets_current += 2;
    }

    opj_write_bytes(p_j2k->m_specific_param.m_encoder.m_tlm_sot_offsets_current,
                    p_tile_part_size, 4);                                       /* PSOT */
    p_j2k->m_specific_param.m_encoder.m_tlm_sot_offsets_current += 4;
}

/**
 * Writes the RGN marker (Region Of Interest)
 *
 * @param       p_tile_no               the tile to output
 * @param       p_comp_no               the component to output
 * @param       nb_comps                the number of components
 * @param       p_stream                the stream to write data to.
 * @param       p_j2k                   J2K codec.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_write_rgn(opj_j2k_t *p_j2k,
                                  OPJ_UINT32 p_tile_no,
                                  OPJ_UINT32 p_comp_no,
                                  OPJ_UINT32 nb_comps,
                                  opj_stream_private_t *p_stream,
                                  opj_event_mgr_t * p_manager);

/**
 * Reads a RGN marker (Region Of Interest)
 *
 * @param       p_header_data   the data contained in the POC box.
 * @param       p_j2k                   the jpeg2000 codec.
 * @param       p_header_size   the size of the data contained in the POC marker.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_read_rgn(opj_j2k_t *p_j2k,
                                 OPJ_BYTE * p_header_data,
                                 OPJ_UINT32 p_header_size,
                                 opj_event_mgr_t * p_manager);

/**
 * Writes the EOC marker (End of Codestream)
 *
 * @param       p_stream                the stream to write data to.
 * @param       p_j2k                   J2K codec.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_write_eoc(opj_j2k_t *p_j2k,
                                  opj_stream_private_t *p_stream,
                                  opj_event_mgr_t * p_manager);

#if 0
/**
 * Reads a EOC marker (End Of Codestream)
 *
 * @param       p_j2k                   the jpeg2000 codec.
 * @param       p_stream                FIXME DOC
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_read_eoc(opj_j2k_t *p_j2k,
                                 opj_stream_private_t *p_stream,
                                 opj_event_mgr_t * p_manager);
#endif

/**
 * Writes the CBD-MCT-MCC-MCO markers (Multi components transform)
 *
 * @param       p_stream                        the stream to write data to.
 * @param       p_j2k                   J2K codec.
 * @param       p_manager       the user event manager.
*/
static OPJ_BOOL opj_j2k_write_mct_data_group(opj_j2k_t *p_j2k,
        opj_stream_private_t *p_stream,
        opj_event_mgr_t * p_manager);

/**
 * Inits the Info
 *
 * @param       p_stream                the stream to write data to.
 * @param       p_j2k                   J2K codec.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_init_info(opj_j2k_t *p_j2k,
                                  opj_stream_private_t *p_stream,
                                  opj_event_mgr_t * p_manager);

/**
Add main header marker information
@param cstr_index    Codestream information structure
@param type         marker type
@param pos          byte offset of marker segment
@param len          length of marker segment
 */
static OPJ_BOOL opj_j2k_add_mhmarker(opj_codestream_index_t *cstr_index,
                                     OPJ_UINT32 type, OPJ_OFF_T pos, OPJ_UINT32 len) ;
/**
Add tile header marker information
@param tileno       tile index number
@param cstr_index   Codestream information structure
@param type         marker type
@param pos          byte offset of marker segment
@param len          length of marker segment
 */
static OPJ_BOOL opj_j2k_add_tlmarker(OPJ_UINT32 tileno,
                                     opj_codestream_index_t *cstr_index, OPJ_UINT32 type, OPJ_OFF_T pos,
                                     OPJ_UINT32 len);

/**
 * Reads an unknown marker
 *
 * @param       p_j2k                   the jpeg2000 codec.
 * @param       p_stream                the stream object to read from.
 * @param       output_marker           FIXME DOC
 * @param       p_manager               the user event manager.
 *
 * @return      true                    if the marker could be deduced.
*/
static OPJ_BOOL opj_j2k_read_unk(opj_j2k_t *p_j2k,
                                 opj_stream_private_t *p_stream,
                                 OPJ_UINT32 *output_marker,
                                 opj_event_mgr_t * p_manager);

/**
 * Writes the MCT marker (Multiple Component Transform)
 *
 * @param       p_j2k           J2K codec.
 * @param       p_mct_record    FIXME DOC
 * @param       p_stream        the stream to write data to.
 * @param       p_manager       the user event manager.
*/
static OPJ_BOOL opj_j2k_write_mct_record(opj_j2k_t *p_j2k,
        opj_mct_data_t * p_mct_record,
        opj_stream_private_t *p_stream,
        opj_event_mgr_t * p_manager);

/**
 * Reads a MCT marker (Multiple Component Transform)
 *
 * @param       p_header_data   the data contained in the MCT box.
 * @param       p_j2k                   the jpeg2000 codec.
 * @param       p_header_size   the size of the data contained in the MCT marker.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_read_mct(opj_j2k_t *p_j2k,
                                 OPJ_BYTE * p_header_data,
                                 OPJ_UINT32 p_header_size,
                                 opj_event_mgr_t * p_manager);

/**
 * Writes the MCC marker (Multiple Component Collection)
 *
 * @param       p_j2k                   J2K codec.
 * @param       p_mcc_record            FIXME DOC
 * @param       p_stream                the stream to write data to.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_write_mcc_record(opj_j2k_t *p_j2k,
        opj_simple_mcc_decorrelation_data_t * p_mcc_record,
        opj_stream_private_t *p_stream,
        opj_event_mgr_t * p_manager);

/**
 * Reads a MCC marker (Multiple Component Collection)
 *
 * @param       p_header_data   the data contained in the MCC box.
 * @param       p_j2k                   the jpeg2000 codec.
 * @param       p_header_size   the size of the data contained in the MCC marker.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_read_mcc(opj_j2k_t *p_j2k,
                                 OPJ_BYTE * p_header_data,
                                 OPJ_UINT32 p_header_size,
                                 opj_event_mgr_t * p_manager);

/**
 * Writes the MCO marker (Multiple component transformation ordering)
 *
 * @param       p_stream                                the stream to write data to.
 * @param       p_j2k                           J2K codec.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_write_mco(opj_j2k_t *p_j2k,
                                  opj_stream_private_t *p_stream,
                                  opj_event_mgr_t * p_manager);

/**
 * Reads a MCO marker (Multiple Component Transform Ordering)
 *
 * @param       p_header_data   the data contained in the MCO box.
 * @param       p_j2k                   the jpeg2000 codec.
 * @param       p_header_size   the size of the data contained in the MCO marker.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_read_mco(opj_j2k_t *p_j2k,
                                 OPJ_BYTE * p_header_data,
                                 OPJ_UINT32 p_header_size,
                                 opj_event_mgr_t * p_manager);

static OPJ_BOOL opj_j2k_add_mct(opj_tcp_t * p_tcp, opj_image_t * p_image,
                                OPJ_UINT32 p_index);

static void  opj_j2k_read_int16_to_float(const void * p_src_data,
        void * p_dest_data, OPJ_UINT32 p_nb_elem);
static void  opj_j2k_read_int32_to_float(const void * p_src_data,
        void * p_dest_data, OPJ_UINT32 p_nb_elem);
static void  opj_j2k_read_float32_to_float(const void * p_src_data,
        void * p_dest_data, OPJ_UINT32 p_nb_elem);
static void  opj_j2k_read_float64_to_float(const void * p_src_data,
        void * p_dest_data, OPJ_UINT32 p_nb_elem);

static void  opj_j2k_read_int16_to_int32(const void * p_src_data,
        void * p_dest_data, OPJ_UINT32 p_nb_elem);
static void  opj_j2k_read_int32_to_int32(const void * p_src_data,
        void * p_dest_data, OPJ_UINT32 p_nb_elem);
static void  opj_j2k_read_float32_to_int32(const void * p_src_data,
        void * p_dest_data, OPJ_UINT32 p_nb_elem);
static void  opj_j2k_read_float64_to_int32(const void * p_src_data,
        void * p_dest_data, OPJ_UINT32 p_nb_elem);

static void  opj_j2k_write_float_to_int16(const void * p_src_data,
        void * p_dest_data, OPJ_UINT32 p_nb_elem);
static void  opj_j2k_write_float_to_int32(const void * p_src_data,
        void * p_dest_data, OPJ_UINT32 p_nb_elem);
static void  opj_j2k_write_float_to_float(const void * p_src_data,
        void * p_dest_data, OPJ_UINT32 p_nb_elem);
static void  opj_j2k_write_float_to_float64(const void * p_src_data,
        void * p_dest_data, OPJ_UINT32 p_nb_elem);

/**
 * Ends the encoding, i.e. frees memory.
 *
 * @param       p_stream                the stream to write data to.
 * @param       p_j2k                   J2K codec.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_end_encoding(opj_j2k_t *p_j2k,
                                     opj_stream_private_t *p_stream,
                                     opj_event_mgr_t * p_manager);

/**
 * Writes the CBD marker (Component bit depth definition)
 *
 * @param       p_stream                                the stream to write data to.
 * @param       p_j2k                           J2K codec.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_write_cbd(opj_j2k_t *p_j2k,
                                  opj_stream_private_t *p_stream,
                                  opj_event_mgr_t * p_manager);

/**
 * Reads a CBD marker (Component bit depth definition)
 * @param       p_header_data   the data contained in the CBD box.
 * @param       p_j2k                   the jpeg2000 codec.
 * @param       p_header_size   the size of the data contained in the CBD marker.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_read_cbd(opj_j2k_t *p_j2k,
                                 OPJ_BYTE * p_header_data,
                                 OPJ_UINT32 p_header_size,
                                 opj_event_mgr_t * p_manager);

/**
 * Reads a CAP marker (extended capabilities definition). Empty implementation.
 * Found in HTJ2K files
 *
 * @param       p_header_data   the data contained in the CAP box.
 * @param       p_j2k                   the jpeg2000 codec.
 * @param       p_header_size   the size of the data contained in the CAP marker.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_read_cap(opj_j2k_t *p_j2k,
                                 OPJ_BYTE * p_header_data,
                                 OPJ_UINT32 p_header_size,
                                 opj_event_mgr_t * p_manager);

/**
 * Reads a CPF marker (corresponding profile). Empty implementation. Found in HTJ2K files
 * @param       p_header_data   the data contained in the CPF box.
 * @param       p_j2k                   the jpeg2000 codec.
 * @param       p_header_size   the size of the data contained in the CPF marker.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_read_cpf(opj_j2k_t *p_j2k,
                                 OPJ_BYTE * p_header_data,
                                 OPJ_UINT32 p_header_size,
                                 opj_event_mgr_t * p_manager);


/**
 * Writes COC marker for each component.
 *
 * @param       p_stream                the stream to write data to.
 * @param       p_j2k                   J2K codec.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_write_all_coc(opj_j2k_t *p_j2k,
                                      opj_stream_private_t *p_stream,
                                      opj_event_mgr_t * p_manager);

/**
 * Writes QCC marker for each component.
 *
 * @param       p_stream                the stream to write data to.
 * @param       p_j2k                   J2K codec.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_write_all_qcc(opj_j2k_t *p_j2k,
                                      opj_stream_private_t *p_stream,
                                      opj_event_mgr_t * p_manager);

/**
 * Writes regions of interests.
 *
 * @param       p_stream                the stream to write data to.
 * @param       p_j2k                   J2K codec.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_write_regions(opj_j2k_t *p_j2k,
                                      opj_stream_private_t *p_stream,
                                      opj_event_mgr_t * p_manager);

/**
 * Writes EPC ????
 *
 * @param       p_stream                the stream to write data to.
 * @param       p_j2k                   J2K codec.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_write_epc(opj_j2k_t *p_j2k,
                                  opj_stream_private_t *p_stream,
                                  opj_event_mgr_t * p_manager);

/**
 * Checks the progression order changes values. Tells of the poc given as input are valid.
 * A nice message is outputted at errors.
 *
 * @param       p_pocs                  the progression order changes.
 * @param       tileno                  the tile number of interest
 * @param       p_nb_pocs               the number of progression order changes.
 * @param       p_nb_resolutions        the number of resolutions.
 * @param       numcomps                the number of components
 * @param       numlayers               the number of layers.
 * @param       p_manager               the user event manager.
 *
 * @return      true if the pocs are valid.
 */
static OPJ_BOOL opj_j2k_check_poc_val(const opj_poc_t *p_pocs,
                                      OPJ_UINT32 tileno,
                                      OPJ_UINT32 p_nb_pocs,
                                      OPJ_UINT32 p_nb_resolutions,
                                      OPJ_UINT32 numcomps,
                                      OPJ_UINT32 numlayers,
                                      opj_event_mgr_t * p_manager);

/**
 * Gets the number of tile parts used for the given change of progression (if any) and the given tile.
 *
 * @param               cp                      the coding parameters.
 * @param               pino            the offset of the given poc (i.e. its position in the coding parameter).
 * @param               tileno          the given tile.
 *
 * @return              the number of tile parts.
 */
static OPJ_UINT32 opj_j2k_get_num_tp(opj_cp_t *cp, OPJ_UINT32 pino,
                                     OPJ_UINT32 tileno);

/**
 * Calculates the total number of tile parts needed by the encoder to
 * encode such an image. If not enough memory is available, then the function return false.
 *
 * @param       p_nb_tiles      pointer that will hold the number of tile parts.
 * @param       cp                      the coding parameters for the image.
 * @param       image           the image to encode.
 * @param       p_j2k                   the p_j2k encoder.
 * @param       p_manager       the user event manager.
 *
 * @return true if the function was successful, false else.
 */
static OPJ_BOOL opj_j2k_calculate_tp(opj_j2k_t *p_j2k,
                                     opj_cp_t *cp,
                                     OPJ_UINT32 * p_nb_tiles,
                                     opj_image_t *image,
                                     opj_event_mgr_t * p_manager);

static void opj_j2k_dump_MH_info(opj_j2k_t* p_j2k, FILE* out_stream);

static void opj_j2k_dump_MH_index(opj_j2k_t* p_j2k, FILE* out_stream);

static opj_codestream_index_t* opj_j2k_create_cstr_index(void);

static OPJ_FLOAT32 opj_j2k_get_tp_stride(opj_tcp_t * p_tcp);

static OPJ_FLOAT32 opj_j2k_get_default_stride(opj_tcp_t * p_tcp);

static int opj_j2k_initialise_4K_poc(opj_poc_t *POC, int numres);

static void opj_j2k_set_cinema_parameters(opj_cparameters_t *parameters,
        opj_image_t *image, opj_event_mgr_t *p_manager);

static OPJ_BOOL opj_j2k_is_cinema_compliant(opj_image_t *image, OPJ_UINT16 rsiz,
        opj_event_mgr_t *p_manager);

static void opj_j2k_set_imf_parameters(opj_cparameters_t *parameters,
                                       opj_image_t *image, opj_event_mgr_t *p_manager);

static OPJ_BOOL opj_j2k_is_imf_compliant(opj_cparameters_t *parameters,
        opj_image_t *image,
        opj_event_mgr_t *p_manager);

/**
 * Checks for invalid number of tile-parts in SOT marker (TPsot==TNsot). See issue 254.
 *
 * @param       p_stream            the stream to read data from.
 * @param       tile_no             tile number we're looking for.
 * @param       p_correction_needed output value. if true, non conformant codestream needs TNsot correction.
 * @param       p_manager       the user event manager.
 *
 * @return true if the function was successful, false else.
 */
static OPJ_BOOL opj_j2k_need_nb_tile_parts_correction(opj_stream_private_t
        *p_stream, OPJ_UINT32 tile_no, OPJ_BOOL* p_correction_needed,
        opj_event_mgr_t * p_manager);

/*@}*/

/*@}*/

/* ----------------------------------------------------------------------- */
typedef struct j2k_prog_order {
    OPJ_PROG_ORDER enum_prog;
    char str_prog[5];
} j2k_prog_order_t;

static const j2k_prog_order_t j2k_prog_order_list[] = {
    {OPJ_CPRL, "CPRL"},
    {OPJ_LRCP, "LRCP"},
    {OPJ_PCRL, "PCRL"},
    {OPJ_RLCP, "RLCP"},
    {OPJ_RPCL, "RPCL"},
    {(OPJ_PROG_ORDER) - 1, ""}
};

/**
 * FIXME DOC
 */
static const OPJ_UINT32 MCT_ELEMENT_SIZE [] = {
    2,
    4,
    4,
    8
};

typedef void (* opj_j2k_mct_function)(const void * p_src_data,
                                      void * p_dest_data, OPJ_UINT32 p_nb_elem);

static const opj_j2k_mct_function j2k_mct_read_functions_to_float [] = {
    opj_j2k_read_int16_to_float,
    opj_j2k_read_int32_to_float,
    opj_j2k_read_float32_to_float,
    opj_j2k_read_float64_to_float
};

static const opj_j2k_mct_function j2k_mct_read_functions_to_int32 [] = {
    opj_j2k_read_int16_to_int32,
    opj_j2k_read_int32_to_int32,
    opj_j2k_read_float32_to_int32,
    opj_j2k_read_float64_to_int32
};

static const opj_j2k_mct_function j2k_mct_write_functions_from_float [] = {
    opj_j2k_write_float_to_int16,
    opj_j2k_write_float_to_int32,
    opj_j2k_write_float_to_float,
    opj_j2k_write_float_to_float64
};

typedef struct opj_dec_memory_marker_handler {
    /** marker value */
    OPJ_UINT32 id;
    /** value of the state when the marker can appear */
    OPJ_UINT32 states;
    /** action linked to the marker */
    OPJ_BOOL(*handler)(opj_j2k_t *p_j2k,
                       OPJ_BYTE * p_header_data,
                       OPJ_UINT32 p_header_size,
                       opj_event_mgr_t * p_manager);
}
opj_dec_memory_marker_handler_t;

static const opj_dec_memory_marker_handler_t j2k_memory_marker_handler_tab [] =
{
    {J2K_MS_SOT, J2K_STATE_MH | J2K_STATE_TPHSOT, opj_j2k_read_sot},
    {J2K_MS_COD, J2K_STATE_MH | J2K_STATE_TPH, opj_j2k_read_cod},
    {J2K_MS_COC, J2K_STATE_MH | J2K_STATE_TPH, opj_j2k_read_coc},
    {J2K_MS_RGN, J2K_STATE_MH | J2K_STATE_TPH, opj_j2k_read_rgn},
    {J2K_MS_QCD, J2K_STATE_MH | J2K_STATE_TPH, opj_j2k_read_qcd},
    {J2K_MS_QCC, J2K_STATE_MH | J2K_STATE_TPH, opj_j2k_read_qcc},
    {J2K_MS_POC, J2K_STATE_MH | J2K_STATE_TPH, opj_j2k_read_poc},
    {J2K_MS_SIZ, J2K_STATE_MHSIZ, opj_j2k_read_siz},
    {J2K_MS_TLM, J2K_STATE_MH, opj_j2k_read_tlm},
    {J2K_MS_PLM, J2K_STATE_MH, opj_j2k_read_plm},
    {J2K_MS_PLT, J2K_STATE_TPH, opj_j2k_read_plt},
    {J2K_MS_PPM, J2K_STATE_MH, opj_j2k_read_ppm},
    {J2K_MS_PPT, J2K_STATE_TPH, opj_j2k_read_ppt},
    {J2K_MS_SOP, 0, 0},
    {J2K_MS_CRG, J2K_STATE_MH, opj_j2k_read_crg},
    {J2K_MS_COM, J2K_STATE_MH | J2K_STATE_TPH, opj_j2k_read_com},
    {J2K_MS_MCT, J2K_STATE_MH | J2K_STATE_TPH, opj_j2k_read_mct},
    {J2K_MS_CBD, J2K_STATE_MH, opj_j2k_read_cbd},
    {J2K_MS_CAP, J2K_STATE_MH, opj_j2k_read_cap},
    {J2K_MS_CPF, J2K_STATE_MH, opj_j2k_read_cpf},
    {J2K_MS_MCC, J2K_STATE_MH | J2K_STATE_TPH, opj_j2k_read_mcc},
    {J2K_MS_MCO, J2K_STATE_MH | J2K_STATE_TPH, opj_j2k_read_mco},
#ifdef USE_JPWL
#ifdef TODO_MS /* remove these functions which are not compatible with the v2 API */
    {J2K_MS_EPC, J2K_STATE_MH | J2K_STATE_TPH, j2k_read_epc},
    {J2K_MS_EPB, J2K_STATE_MH | J2K_STATE_TPH, j2k_read_epb},
    {J2K_MS_ESD, J2K_STATE_MH | J2K_STATE_TPH, j2k_read_esd},
    {J2K_MS_RED, J2K_STATE_MH | J2K_STATE_TPH, j2k_read_red},
#endif
#endif /* USE_JPWL */
#ifdef USE_JPSEC
    {J2K_MS_SEC, J2K_DEC_STATE_MH, j2k_read_sec},
    {J2K_MS_INSEC, 0, j2k_read_insec}
#endif /* USE_JPSEC */
    {J2K_MS_UNK, J2K_STATE_MH | J2K_STATE_TPH, 0}/*opj_j2k_read_unk is directly used*/
};

static void  opj_j2k_read_int16_to_float(const void * p_src_data,
        void * p_dest_data, OPJ_UINT32 p_nb_elem)
{
    OPJ_BYTE * l_src_data = (OPJ_BYTE *) p_src_data;
    OPJ_FLOAT32 * l_dest_data = (OPJ_FLOAT32 *) p_dest_data;
    OPJ_UINT32 i;
    OPJ_UINT32 l_temp;

    for (i = 0; i < p_nb_elem; ++i) {
        opj_read_bytes(l_src_data, &l_temp, 2);

        l_src_data += sizeof(OPJ_INT16);

        *(l_dest_data++) = (OPJ_FLOAT32) l_temp;
    }
}

static void  opj_j2k_read_int32_to_float(const void * p_src_data,
        void * p_dest_data, OPJ_UINT32 p_nb_elem)
{
    OPJ_BYTE * l_src_data = (OPJ_BYTE *) p_src_data;
    OPJ_FLOAT32 * l_dest_data = (OPJ_FLOAT32 *) p_dest_data;
    OPJ_UINT32 i;
    OPJ_UINT32 l_temp;

    for (i = 0; i < p_nb_elem; ++i) {
        opj_read_bytes(l_src_data, &l_temp, 4);

        l_src_data += sizeof(OPJ_INT32);

        *(l_dest_data++) = (OPJ_FLOAT32) l_temp;
    }
}

static void  opj_j2k_read_float32_to_float(const void * p_src_data,
        void * p_dest_data, OPJ_UINT32 p_nb_elem)
{
    OPJ_BYTE * l_src_data = (OPJ_BYTE *) p_src_data;
    OPJ_FLOAT32 * l_dest_data = (OPJ_FLOAT32 *) p_dest_data;
    OPJ_UINT32 i;
    OPJ_FLOAT32 l_temp;

    for (i = 0; i < p_nb_elem; ++i) {
        opj_read_float(l_src_data, &l_temp);

        l_src_data += sizeof(OPJ_FLOAT32);

        *(l_dest_data++) = l_temp;
    }
}

static void  opj_j2k_read_float64_to_float(const void * p_src_data,
        void * p_dest_data, OPJ_UINT32 p_nb_elem)
{
    OPJ_BYTE * l_src_data = (OPJ_BYTE *) p_src_data;
    OPJ_FLOAT32 * l_dest_data = (OPJ_FLOAT32 *) p_dest_data;
    OPJ_UINT32 i;
    OPJ_FLOAT64 l_temp;

    for (i = 0; i < p_nb_elem; ++i) {
        opj_read_double(l_src_data, &l_temp);

        l_src_data += sizeof(OPJ_FLOAT64);

        *(l_dest_data++) = (OPJ_FLOAT32) l_temp;
    }
}

static void  opj_j2k_read_int16_to_int32(const void * p_src_data,
        void * p_dest_data, OPJ_UINT32 p_nb_elem)
{
    OPJ_BYTE * l_src_data = (OPJ_BYTE *) p_src_data;
    OPJ_INT32 * l_dest_data = (OPJ_INT32 *) p_dest_data;
    OPJ_UINT32 i;
    OPJ_UINT32 l_temp;

    for (i = 0; i < p_nb_elem; ++i) {
        opj_read_bytes(l_src_data, &l_temp, 2);

        l_src_data += sizeof(OPJ_INT16);

        *(l_dest_data++) = (OPJ_INT32) l_temp;
    }
}

static void  opj_j2k_read_int32_to_int32(const void * p_src_data,
        void * p_dest_data, OPJ_UINT32 p_nb_elem)
{
    OPJ_BYTE * l_src_data = (OPJ_BYTE *) p_src_data;
    OPJ_INT32 * l_dest_data = (OPJ_INT32 *) p_dest_data;
    OPJ_UINT32 i;
    OPJ_UINT32 l_temp;

    for (i = 0; i < p_nb_elem; ++i) {
        opj_read_bytes(l_src_data, &l_temp, 4);

        l_src_data += sizeof(OPJ_INT32);

        *(l_dest_data++) = (OPJ_INT32) l_temp;
    }
}

static void  opj_j2k_read_float32_to_int32(const void * p_src_data,
        void * p_dest_data, OPJ_UINT32 p_nb_elem)
{
    OPJ_BYTE * l_src_data = (OPJ_BYTE *) p_src_data;
    OPJ_INT32 * l_dest_data = (OPJ_INT32 *) p_dest_data;
    OPJ_UINT32 i;
    OPJ_FLOAT32 l_temp;

    for (i = 0; i < p_nb_elem; ++i) {
        opj_read_float(l_src_data, &l_temp);

        l_src_data += sizeof(OPJ_FLOAT32);

        *(l_dest_data++) = (OPJ_INT32) l_temp;
    }
}

static void  opj_j2k_read_float64_to_int32(const void * p_src_data,
        void * p_dest_data, OPJ_UINT32 p_nb_elem)
{
    OPJ_BYTE * l_src_data = (OPJ_BYTE *) p_src_data;
    OPJ_INT32 * l_dest_data = (OPJ_INT32 *) p_dest_data;
    OPJ_UINT32 i;
    OPJ_FLOAT64 l_temp;

    for (i = 0; i < p_nb_elem; ++i) {
        opj_read_double(l_src_data, &l_temp);

        l_src_data += sizeof(OPJ_FLOAT64);

        *(l_dest_data++) = (OPJ_INT32) l_temp;
    }
}

static void  opj_j2k_write_float_to_int16(const void * p_src_data,
        void * p_dest_data, OPJ_UINT32 p_nb_elem)
{
    OPJ_BYTE * l_dest_data = (OPJ_BYTE *) p_dest_data;
    OPJ_FLOAT32 * l_src_data = (OPJ_FLOAT32 *) p_src_data;
    OPJ_UINT32 i;
    OPJ_UINT32 l_temp;

    for (i = 0; i < p_nb_elem; ++i) {
        l_temp = (OPJ_UINT32) * (l_src_data++);

        opj_write_bytes(l_dest_data, l_temp, sizeof(OPJ_INT16));

        l_dest_data += sizeof(OPJ_INT16);
    }
}

static void opj_j2k_write_float_to_int32(const void * p_src_data,
        void * p_dest_data, OPJ_UINT32 p_nb_elem)
{
    OPJ_BYTE * l_dest_data = (OPJ_BYTE *) p_dest_data;
    OPJ_FLOAT32 * l_src_data = (OPJ_FLOAT32 *) p_src_data;
    OPJ_UINT32 i;
    OPJ_UINT32 l_temp;

    for (i = 0; i < p_nb_elem; ++i) {
        l_temp = (OPJ_UINT32) * (l_src_data++);

        opj_write_bytes(l_dest_data, l_temp, sizeof(OPJ_INT32));

        l_dest_data += sizeof(OPJ_INT32);
    }
}

static void  opj_j2k_write_float_to_float(const void * p_src_data,
        void * p_dest_data, OPJ_UINT32 p_nb_elem)
{
    OPJ_BYTE * l_dest_data = (OPJ_BYTE *) p_dest_data;
    OPJ_FLOAT32 * l_src_data = (OPJ_FLOAT32 *) p_src_data;
    OPJ_UINT32 i;
    OPJ_FLOAT32 l_temp;

    for (i = 0; i < p_nb_elem; ++i) {
        l_temp = (OPJ_FLOAT32) * (l_src_data++);

        opj_write_float(l_dest_data, l_temp);

        l_dest_data += sizeof(OPJ_FLOAT32);
    }
}

static void  opj_j2k_write_float_to_float64(const void * p_src_data,
        void * p_dest_data, OPJ_UINT32 p_nb_elem)
{
    OPJ_BYTE * l_dest_data = (OPJ_BYTE *) p_dest_data;
    OPJ_FLOAT32 * l_src_data = (OPJ_FLOAT32 *) p_src_data;
    OPJ_UINT32 i;
    OPJ_FLOAT64 l_temp;

    for (i = 0; i < p_nb_elem; ++i) {
        l_temp = (OPJ_FLOAT64) * (l_src_data++);

        opj_write_double(l_dest_data, l_temp);

        l_dest_data += sizeof(OPJ_FLOAT64);
    }
}

const char *opj_j2k_convert_progression_order(OPJ_PROG_ORDER prg_order)
{
    const j2k_prog_order_t *po;
    for (po = j2k_prog_order_list; po->enum_prog != -1; po++) {
        if (po->enum_prog == prg_order) {
            return po->str_prog;
        }
    }
    return po->str_prog;
}

static OPJ_BOOL opj_j2k_check_poc_val(const opj_poc_t *p_pocs,
                                      OPJ_UINT32 tileno,
                                      OPJ_UINT32 p_nb_pocs,
                                      OPJ_UINT32 p_nb_resolutions,
                                      OPJ_UINT32 p_num_comps,
                                      OPJ_UINT32 p_num_layers,
                                      opj_event_mgr_t * p_manager)
{
    OPJ_UINT32* packet_array;
    OPJ_UINT32 index, resno, compno, layno;
    OPJ_UINT32 i;
    OPJ_UINT32 step_c = 1;
    OPJ_UINT32 step_r = p_num_comps * step_c;
    OPJ_UINT32 step_l = p_nb_resolutions * step_r;
    OPJ_BOOL loss = OPJ_FALSE;

    assert(p_nb_pocs > 0);

    packet_array = (OPJ_UINT32*) opj_calloc((size_t)step_l * p_num_layers,
                                            sizeof(OPJ_UINT32));
    if (packet_array == 00) {
        opj_event_msg(p_manager, EVT_ERROR,
                      "Not enough memory for checking the poc values.\n");
        return OPJ_FALSE;
    }

    /* iterate through all the pocs that match our tile of interest. */
    for (i = 0; i < p_nb_pocs; ++i) {
        const opj_poc_t *poc = &p_pocs[i];
        if (tileno + 1 == poc->tile) {
            index = step_r * poc->resno0;

            /* take each resolution for each poc */
            for (resno = poc->resno0 ;
                    resno < opj_uint_min(poc->resno1, p_nb_resolutions); ++resno) {
                OPJ_UINT32 res_index = index + poc->compno0 * step_c;

                /* take each comp of each resolution for each poc */
                for (compno = poc->compno0 ;
                        compno < opj_uint_min(poc->compno1, p_num_comps); ++compno) {
                    /* The layer index always starts at zero for every progression. */
                    const OPJ_UINT32 layno0 = 0;
                    OPJ_UINT32 comp_index = res_index + layno0 * step_l;

                    /* and finally take each layer of each res of ... */
                    for (layno = layno0; layno < opj_uint_min(poc->layno1, p_num_layers);
                            ++layno) {
                        packet_array[comp_index] = 1;
                        comp_index += step_l;
                    }

                    res_index += step_c;
                }

                index += step_r;
            }
        }
    }

    index = 0;
    for (layno = 0; layno < p_num_layers ; ++layno) {
        for (resno = 0; resno < p_nb_resolutions; ++resno) {
            for (compno = 0; compno < p_num_comps; ++compno) {
                loss |= (packet_array[index] != 1);
#ifdef DEBUG_VERBOSE
                if (packet_array[index] != 1) {
                    fprintf(stderr,
                            "Missing packet in POC: layno=%d resno=%d compno=%d\n",
                            layno, resno, compno);
                }
#endif
                index += step_c;
            }
        }
    }

    if (loss) {
        opj_event_msg(p_manager, EVT_ERROR, "Missing packets possible loss of data\n");
    }

    opj_free(packet_array);

    return !loss;
}

/* ----------------------------------------------------------------------- */

static OPJ_UINT32 opj_j2k_get_num_tp(opj_cp_t *cp, OPJ_UINT32 pino,
                                     OPJ_UINT32 tileno)
{
    const OPJ_CHAR *prog = 00;
    OPJ_INT32 i;
    OPJ_UINT32 tpnum = 1;
    opj_tcp_t *tcp = 00;
    opj_poc_t * l_current_poc = 00;

    /*  preconditions */
    assert(tileno < (cp->tw * cp->th));
    assert(pino < (cp->tcps[tileno].numpocs + 1));

    /* get the given tile coding parameter */
    tcp = &cp->tcps[tileno];
    assert(tcp != 00);

    l_current_poc = &(tcp->pocs[pino]);
    assert(l_current_poc != 0);

    /* get the progression order as a character string */
    prog = opj_j2k_convert_progression_order(tcp->prg);
    assert(strlen(prog) > 0);

    if (cp->m_specific_param.m_enc.m_tp_on == 1) {
        for (i = 0; i < 4; ++i) {
            switch (prog[i]) {
            /* component wise */
            case 'C':
                tpnum *= l_current_poc->compE;
                break;
            /* resolution wise */
            case 'R':
                tpnum *= l_current_poc->resE;
                break;
            /* precinct wise */
            case 'P':
                tpnum *= l_current_poc->prcE;
                break;
            /* layer wise */
            case 'L':
                tpnum *= l_current_poc->layE;
                break;
            }
            /* would we split here ? */
            if (cp->m_specific_param.m_enc.m_tp_flag == prog[i]) {
                cp->m_specific_param.m_enc.m_tp_pos = i;
                break;
            }
        }
    } else {
        tpnum = 1;
    }

    return tpnum;
}

static OPJ_BOOL opj_j2k_calculate_tp(opj_j2k_t *p_j2k,
                                     opj_cp_t *cp,
                                     OPJ_UINT32 * p_nb_tiles,
                                     opj_image_t *image,
                                     opj_event_mgr_t * p_manager
                                    )
{
    OPJ_UINT32 pino, tileno;
    OPJ_UINT32 l_nb_tiles;
    opj_tcp_t *tcp;

    /* preconditions */
    assert(p_nb_tiles != 00);
    assert(cp != 00);
    assert(image != 00);
    assert(p_j2k != 00);
    assert(p_manager != 00);

    OPJ_UNUSED(p_j2k);
    OPJ_UNUSED(p_manager);

    l_nb_tiles = cp->tw * cp->th;
    * p_nb_tiles = 0;
    tcp = cp->tcps;

    /* INDEX >> */
    /* TODO mergeV2: check this part which use cstr_info */
    /*if (p_j2k->cstr_info) {
            opj_tile_info_t * l_info_tile_ptr = p_j2k->cstr_info->tile;

            for (tileno = 0; tileno < l_nb_tiles; ++tileno) {
                    OPJ_UINT32 cur_totnum_tp = 0;

                    opj_pi_update_encoding_parameters(image,cp,tileno);

                    for (pino = 0; pino <= tcp->numpocs; ++pino)
                    {
                            OPJ_UINT32 tp_num = opj_j2k_get_num_tp(cp,pino,tileno);

                            *p_nb_tiles = *p_nb_tiles + tp_num;

                            cur_totnum_tp += tp_num;
                    }

                    tcp->m_nb_tile_parts = cur_totnum_tp;

                    l_info_tile_ptr->tp = (opj_tp_info_t *) opj_malloc(cur_totnum_tp * sizeof(opj_tp_info_t));
                    if (l_info_tile_ptr->tp == 00) {
                            return OPJ_FALSE;
                    }

                    memset(l_info_tile_ptr->tp,0,cur_totnum_tp * sizeof(opj_tp_info_t));

                    l_info_tile_ptr->num_tps = cur_totnum_tp;

                    ++l_info_tile_ptr;
                    ++tcp;
            }
    }
    else */{
        for (tileno = 0; tileno < l_nb_tiles; ++tileno) {
            OPJ_UINT32 cur_totnum_tp = 0;

            opj_pi_update_encoding_parameters(image, cp, tileno);

            for (pino = 0; pino <= tcp->numpocs; ++pino) {
                OPJ_UINT32 tp_num = opj_j2k_get_num_tp(cp, pino, tileno);

                *p_nb_tiles = *p_nb_tiles + tp_num;

                cur_totnum_tp += tp_num;
            }
            tcp->m_nb_tile_parts = cur_totnum_tp;

            ++tcp;
        }
    }

    return OPJ_TRUE;
}

static OPJ_BOOL opj_j2k_write_soc(opj_j2k_t *p_j2k,
                                  opj_stream_private_t *p_stream,
                                  opj_event_mgr_t * p_manager)
{
    /* 2 bytes will be written */
    OPJ_BYTE * l_start_stream = 00;

    /* preconditions */
    assert(p_stream != 00);
    assert(p_j2k != 00);
    assert(p_manager != 00);

    l_start_stream = p_j2k->m_specific_param.m_encoder.m_header_tile_data;

    /* write SOC identifier */
    opj_write_bytes(l_start_stream, J2K_MS_SOC, 2);

    if (opj_stream_write_data(p_stream, l_start_stream, 2, p_manager) != 2) {
        return OPJ_FALSE;
    }

    /* UniPG>> */
#ifdef USE_JPWL
    /* update markers struct */
    /*
            OPJ_BOOL res = j2k_add_marker(p_j2k->cstr_info, J2K_MS_SOC, p_stream_tell(p_stream) - 2, 2);
    */
    assert(0 && "TODO");
#endif /* USE_JPWL */
    /* <<UniPG */

    return OPJ_TRUE;
}

/**
 * Reads a SOC marker (Start of Codestream)
 * @param       p_j2k           the jpeg2000 file codec.
 * @param       p_stream        FIXME DOC
 * @param       p_manager       the user event manager.
*/
static OPJ_BOOL opj_j2k_read_soc(opj_j2k_t *p_j2k,
                                 opj_stream_private_t *p_stream,
                                 opj_event_mgr_t * p_manager
                                )
{
    OPJ_BYTE l_data [2];
    OPJ_UINT32 l_marker;

    /* preconditions */
    assert(p_j2k != 00);
    assert(p_manager != 00);
    assert(p_stream != 00);

    if (opj_stream_read_data(p_stream, l_data, 2, p_manager) != 2) {
        return OPJ_FALSE;
    }

    opj_read_bytes(l_data, &l_marker, 2);
    if (l_marker != J2K_MS_SOC) {
        return OPJ_FALSE;
    }

    /* Next marker should be a SIZ marker in the main header */
    p_j2k->m_specific_param.m_decoder.m_state = J2K_STATE_MHSIZ;

    /* FIXME move it in a index structure included in p_j2k*/
    p_j2k->cstr_index->main_head_start = opj_stream_tell(p_stream) - 2;

    opj_event_msg(p_manager, EVT_INFO,
                  "Start to read j2k main header (%" PRId64 ").\n",
                  p_j2k->cstr_index->main_head_start);

    /* Add the marker to the codestream index*/
    if (OPJ_FALSE == opj_j2k_add_mhmarker(p_j2k->cstr_index, J2K_MS_SOC,
                                          p_j2k->cstr_index->main_head_start, 2)) {
        opj_event_msg(p_manager, EVT_ERROR, "Not enough memory to add mh marker\n");
        return OPJ_FALSE;
    }
    return OPJ_TRUE;
}

static OPJ_BOOL opj_j2k_write_siz(opj_j2k_t *p_j2k,
                                  opj_stream_private_t *p_stream,
                                  opj_event_mgr_t * p_manager)
{
    OPJ_UINT32 i;
    OPJ_UINT32 l_size_len;
    OPJ_BYTE * l_current_ptr;
    opj_image_t * l_image = 00;
    opj_cp_t *cp = 00;
    opj_image_comp_t * l_img_comp = 00;

    /* preconditions */
    assert(p_stream != 00);
    assert(p_j2k != 00);
    assert(p_manager != 00);

    l_image = p_j2k->m_private_image;
    cp = &(p_j2k->m_cp);
    l_size_len = 40 + 3 * l_image->numcomps;
    l_img_comp = l_image->comps;

    if (l_size_len > p_j2k->m_specific_param.m_encoder.m_header_tile_data_size) {

        OPJ_BYTE *new_header_tile_data = (OPJ_BYTE *) opj_realloc(
                                             p_j2k->m_specific_param.m_encoder.m_header_tile_data, l_size_len);
        if (! new_header_tile_data) {
            opj_free(p_j2k->m_specific_param.m_encoder.m_header_tile_data);
            p_j2k->m_specific_param.m_encoder.m_header_tile_data = NULL;
            p_j2k->m_specific_param.m_encoder.m_header_tile_data_size = 0;
            opj_event_msg(p_manager, EVT_ERROR, "Not enough memory for the SIZ marker\n");
            return OPJ_FALSE;
        }
        p_j2k->m_specific_param.m_encoder.m_header_tile_data = new_header_tile_data;
        p_j2k->m_specific_param.m_encoder.m_header_tile_data_size = l_size_len;
    }

    l_current_ptr = p_j2k->m_specific_param.m_encoder.m_header_tile_data;

    /* write SOC identifier */
    opj_write_bytes(l_current_ptr, J2K_MS_SIZ, 2);  /* SIZ */
    l_current_ptr += 2;

    opj_write_bytes(l_current_ptr, l_size_len - 2, 2); /* L_SIZ */
    l_current_ptr += 2;

    opj_write_bytes(l_current_ptr, cp->rsiz, 2);    /* Rsiz (capabilities) */
    l_current_ptr += 2;

    opj_write_bytes(l_current_ptr, l_image->x1, 4); /* Xsiz */
    l_current_ptr += 4;

    opj_write_bytes(l_current_ptr, l_image->y1, 4); /* Ysiz */
    l_current_ptr += 4;

    opj_write_bytes(l_current_ptr, l_image->x0, 4); /* X0siz */
    l_current_ptr += 4;

    opj_write_bytes(l_current_ptr, l_image->y0, 4); /* Y0siz */
    l_current_ptr += 4;

    opj_write_bytes(l_current_ptr, cp->tdx, 4);             /* XTsiz */
    l_current_ptr += 4;

    opj_write_bytes(l_current_ptr, cp->tdy, 4);             /* YTsiz */
    l_current_ptr += 4;

    opj_write_bytes(l_current_ptr, cp->tx0, 4);             /* XT0siz */
    l_current_ptr += 4;

    opj_write_bytes(l_current_ptr, cp->ty0, 4);             /* YT0siz */
    l_current_ptr += 4;

    opj_write_bytes(l_current_ptr, l_image->numcomps, 2);   /* Csiz */
    l_current_ptr += 2;

    for (i = 0; i < l_image->numcomps; ++i) {
        /* TODO here with MCT ? */
        opj_write_bytes(l_current_ptr, l_img_comp->prec - 1 + (l_img_comp->sgnd << 7),
                        1);      /* Ssiz_i */
        ++l_current_ptr;

        opj_write_bytes(l_current_ptr, l_img_comp->dx, 1);      /* XRsiz_i */
        ++l_current_ptr;

        opj_write_bytes(l_current_ptr, l_img_comp->dy, 1);      /* YRsiz_i */
        ++l_current_ptr;

        ++l_img_comp;
    }

    if (opj_stream_write_data(p_stream,
                              p_j2k->m_specific_param.m_encoder.m_header_tile_data, l_size_len,
                              p_manager) != l_size_len) {
        return OPJ_FALSE;
    }

    return OPJ_TRUE;
}

/**
 * Reads a SIZ marker (image and tile size)
 * @param       p_j2k           the jpeg2000 file codec.
 * @param       p_header_data   the data contained in the SIZ box.
 * @param       p_header_size   the size of the data contained in the SIZ marker.
 * @param       p_manager       the user event manager.
*/
static OPJ_BOOL opj_j2k_read_siz(opj_j2k_t *p_j2k,
                                 OPJ_BYTE * p_header_data,
                                 OPJ_UINT32 p_header_size,
                                 opj_event_mgr_t * p_manager
                                )
{
    OPJ_UINT32 i;
    OPJ_UINT32 l_nb_comp;
    OPJ_UINT32 l_nb_comp_remain;
    OPJ_UINT32 l_remaining_size;
    OPJ_UINT32 l_nb_tiles;
    OPJ_UINT32 l_tmp, l_tx1, l_ty1;
    OPJ_UINT32 l_prec0, l_sgnd0;
    opj_image_t *l_image = 00;
    opj_cp_t *l_cp = 00;
    opj_image_comp_t * l_img_comp = 00;
    opj_tcp_t * l_current_tile_param = 00;

    /* preconditions */
    assert(p_j2k != 00);
    assert(p_manager != 00);
    assert(p_header_data != 00);

    l_image = p_j2k->m_private_image;
    l_cp = &(p_j2k->m_cp);

    /* minimum size == 39 - 3 (= minimum component parameter) */
    if (p_header_size < 36) {
        opj_event_msg(p_manager, EVT_ERROR, "Error with SIZ marker size\n");
        return OPJ_FALSE;
    }

    l_remaining_size = p_header_size - 36;
    l_nb_comp = l_remaining_size / 3;
    l_nb_comp_remain = l_remaining_size % 3;
    if (l_nb_comp_remain != 0) {
        opj_event_msg(p_manager, EVT_ERROR, "Error with SIZ marker size\n");
        return OPJ_FALSE;
    }

    opj_read_bytes(p_header_data, &l_tmp,
                   2);                                                /* Rsiz (capabilities) */
    p_header_data += 2;
    l_cp->rsiz = (OPJ_UINT16) l_tmp;
    opj_read_bytes(p_header_data, (OPJ_UINT32*) &l_image->x1, 4);   /* Xsiz */
    p_header_data += 4;
    opj_read_bytes(p_header_data, (OPJ_UINT32*) &l_image->y1, 4);   /* Ysiz */
    p_header_data += 4;
    opj_read_bytes(p_header_data, (OPJ_UINT32*) &l_image->x0, 4);   /* X0siz */
    p_header_data += 4;
    opj_read_bytes(p_header_data, (OPJ_UINT32*) &l_image->y0, 4);   /* Y0siz */
    p_header_data += 4;
    opj_read_bytes(p_header_data, (OPJ_UINT32*) &l_cp->tdx,
                   4);             /* XTsiz */
    p_header_data += 4;
    opj_read_bytes(p_header_data, (OPJ_UINT32*) &l_cp->tdy,
                   4);             /* YTsiz */
    p_header_data += 4;
    opj_read_bytes(p_header_data, (OPJ_UINT32*) &l_cp->tx0,
                   4);             /* XT0siz */
    p_header_data += 4;
    opj_read_bytes(p_header_data, (OPJ_UINT32*) &l_cp->ty0,
                   4);             /* YT0siz */
    p_header_data += 4;
    opj_read_bytes(p_header_data, (OPJ_UINT32*) &l_tmp,
                   2);                 /* Csiz */
    p_header_data += 2;
    if (l_tmp < 16385) {
        l_image->numcomps = (OPJ_UINT16) l_tmp;
    } else {
        opj_event_msg(p_manager, EVT_ERROR,
                      "Error with SIZ marker: number of component is illegal -> %d\n", l_tmp);
        return OPJ_FALSE;
    }

    if (l_image->numcomps != l_nb_comp) {
        opj_event_msg(p_manager, EVT_ERROR,
                      "Error with SIZ marker: number of component is not compatible with the remaining number of parameters ( %d vs %d)\n",
                      l_image->numcomps, l_nb_comp);
        return OPJ_FALSE;
    }

    /* testcase 4035.pdf.SIGSEGV.d8b.3375 */
    /* testcase issue427-null-image-size.jp2 */
    if ((l_image->x0 >= l_image->x1) || (l_image->y0 >= l_image->y1)) {
        opj_event_msg(p_manager, EVT_ERROR,
                      "Error with SIZ marker: negative or zero image size (%" PRId64 " x %" PRId64
                      ")\n", (OPJ_INT64)l_image->x1 - l_image->x0,
                      (OPJ_INT64)l_image->y1 - l_image->y0);
        return OPJ_FALSE;
    }
    /* testcase 2539.pdf.SIGFPE.706.1712 (also 3622.pdf.SIGFPE.706.2916 and 4008.pdf.SIGFPE.706.3345 and maybe more) */
    if ((l_cp->tdx == 0U) || (l_cp->tdy == 0U)) {
        opj_event_msg(p_manager, EVT_ERROR,
                      "Error with SIZ marker: invalid tile size (tdx: %d, tdy: %d)\n", l_cp->tdx,
                      l_cp->tdy);
        return OPJ_FALSE;
    }

    /* testcase issue427-illegal-tile-offset.jp2 */
    l_tx1 = opj_uint_adds(l_cp->tx0, l_cp->tdx); /* manage overflow */
    l_ty1 = opj_uint_adds(l_cp->ty0, l_cp->tdy); /* manage overflow */
    if ((l_cp->tx0 > l_image->x0) || (l_cp->ty0 > l_image->y0) ||
            (l_tx1 <= l_image->x0) || (l_ty1 <= l_image->y0)) {
        opj_event_msg(p_manager, EVT_ERROR,
                      "Error with SIZ marker: illegal tile offset\n");
        return OPJ_FALSE;
    }
    if (!p_j2k->dump_state) {
        OPJ_UINT32 siz_w, siz_h;

        siz_w = l_image->x1 - l_image->x0;
        siz_h = l_image->y1 - l_image->y0;

        if (p_j2k->ihdr_w > 0 && p_j2k->ihdr_h > 0
                && (p_j2k->ihdr_w != siz_w || p_j2k->ihdr_h != siz_h)) {
            opj_event_msg(p_manager, EVT_ERROR,
                          "Error with SIZ marker: IHDR w(%u) h(%u) vs. SIZ w(%u) h(%u)\n", p_j2k->ihdr_w,
                          p_j2k->ihdr_h, siz_w, siz_h);
            return OPJ_FALSE;
        }
    }
#ifdef USE_JPWL
    if (l_cp->correct) {
        /* if JPWL is on, we check whether TX errors have damaged
          too much the SIZ parameters */
        if (!(l_image->x1 * l_image->y1)) {
            opj_event_msg(p_manager, EVT_ERROR,
                          "JPWL: bad image size (%d x %d)\n",
                          l_image->x1, l_image->y1);
            if (!JPWL_ASSUME) {
                opj_event_msg(p_manager, EVT_ERROR, "JPWL: giving up\n");
                return OPJ_FALSE;
            }
        }

        /* FIXME check previously in the function so why keep this piece of code ? Need by the norm ?
                if (l_image->numcomps != ((len - 38) / 3)) {
                        opj_event_msg(p_manager, JPWL_ASSUME ? EVT_WARNING : EVT_ERROR,
                                "JPWL: Csiz is %d => space in SIZ only for %d comps.!!!\n",
                                l_image->numcomps, ((len - 38) / 3));
                        if (!JPWL_ASSUME) {
                                opj_event_msg(p_manager, EVT_ERROR, "JPWL: giving up\n");
                                return OPJ_FALSE;
                        }
        */              /* we try to correct */
        /*              opj_event_msg(p_manager, EVT_WARNING, "- trying to adjust this\n");
                        if (l_image->numcomps < ((len - 38) / 3)) {
                                len = 38 + 3 * l_image->numcomps;
                                opj_event_msg(p_manager, EVT_WARNING, "- setting Lsiz to %d => HYPOTHESIS!!!\n",
                                        len);
                        } else {
                                l_image->numcomps = ((len - 38) / 3);
                                opj_event_msg(p_manager, EVT_WARNING, "- setting Csiz to %d => HYPOTHESIS!!!\n",
                                        l_image->numcomps);
                        }
                }
        */

        /* update components number in the jpwl_exp_comps filed */
        l_cp->exp_comps = l_image->numcomps;
    }
#endif /* USE_JPWL */

    /* Allocate the resulting image components */
    l_image->comps = (opj_image_comp_t*) opj_calloc(l_image->numcomps,
                     sizeof(opj_image_comp_t));
    if (l_image->comps == 00) {
        l_image->numcomps = 0;
        opj_event_msg(p_manager, EVT_ERROR,
                      "Not enough memory to take in charge SIZ marker\n");
        return OPJ_FALSE;
    }

    l_img_comp = l_image->comps;

    l_prec0 = 0;
    l_sgnd0 = 0;
    /* Read the component information */
    for (i = 0; i < l_image->numcomps; ++i) {
        OPJ_UINT32 tmp;
        opj_read_bytes(p_header_data, &tmp, 1); /* Ssiz_i */
        ++p_header_data;
        l_img_comp->prec = (tmp & 0x7f) + 1;
        l_img_comp->sgnd = tmp >> 7;

        if (p_j2k->dump_state == 0) {
            if (i == 0) {
                l_prec0 = l_img_comp->prec;
                l_sgnd0 = l_img_comp->sgnd;
            } else if (!l_cp->allow_different_bit_depth_sign
                       && (l_img_comp->prec != l_prec0 || l_img_comp->sgnd != l_sgnd0)) {
                opj_event_msg(p_manager, EVT_WARNING,
                              "Despite JP2 BPC!=255, precision and/or sgnd values for comp[%d] is different than comp[0]:\n"
                              "        [0] prec(%d) sgnd(%d) [%d] prec(%d) sgnd(%d)\n", i, l_prec0, l_sgnd0,
                              i, l_img_comp->prec, l_img_comp->sgnd);
            }
            /* TODO: we should perhaps also check against JP2 BPCC values */
        }
        opj_read_bytes(p_header_data, &tmp, 1); /* XRsiz_i */
        ++p_header_data;
        l_img_comp->dx = (OPJ_UINT32)tmp; /* should be between 1 and 255 */
        opj_read_bytes(p_header_data, &tmp, 1); /* YRsiz_i */
        ++p_header_data;
        l_img_comp->dy = (OPJ_UINT32)tmp; /* should be between 1 and 255 */
        if (l_img_comp->dx < 1 || l_img_comp->dx > 255 ||
                l_img_comp->dy < 1 || l_img_comp->dy > 255) {
            opj_event_msg(p_manager, EVT_ERROR,
                          "Invalid values for comp = %d : dx=%u dy=%u (should be between 1 and 255 according to the JPEG2000 norm)\n",
                          i, l_img_comp->dx, l_img_comp->dy);
            return OPJ_FALSE;
        }
        /* Avoids later undefined shift in computation of */
        /* p_j2k->m_specific_param.m_decoder.m_default_tcp->tccps[i].m_dc_level_shift = 1
                    << (l_image->comps[i].prec - 1); */
        if (l_img_comp->prec > 31) {
            opj_event_msg(p_manager, EVT_ERROR,
                          "Invalid values for comp = %d : prec=%u (should be between 1 and 38 according to the JPEG2000 norm. OpenJpeg only supports up to 31)\n",
                          i, l_img_comp->prec);
            return OPJ_FALSE;
        }
#ifdef USE_JPWL
        if (l_cp->correct) {
            /* if JPWL is on, we check whether TX errors have damaged
                    too much the SIZ parameters, again */
            if (!(l_image->comps[i].dx * l_image->comps[i].dy)) {
                opj_event_msg(p_manager, JPWL_ASSUME ? EVT_WARNING : EVT_ERROR,
                              "JPWL: bad XRsiz_%d/YRsiz_%d (%d x %d)\n",
                              i, i, l_image->comps[i].dx, l_image->comps[i].dy);
                if (!JPWL_ASSUME) {
                    opj_event_msg(p_manager, EVT_ERROR, "JPWL: giving up\n");
                    return OPJ_FALSE;
                }
                /* we try to correct */
                opj_event_msg(p_manager, EVT_WARNING, "- trying to adjust them\n");
                if (!l_image->comps[i].dx) {
                    l_image->comps[i].dx = 1;
                    opj_event_msg(p_manager, EVT_WARNING,
                                  "- setting XRsiz_%d to %d => HYPOTHESIS!!!\n",
                                  i, l_image->comps[i].dx);
                }
                if (!l_image->comps[i].dy) {
                    l_image->comps[i].dy = 1;
                    opj_event_msg(p_manager, EVT_WARNING,
                                  "- setting YRsiz_%d to %d => HYPOTHESIS!!!\n",
                                  i, l_image->comps[i].dy);
                }
            }
        }
#endif /* USE_JPWL */
        l_img_comp->resno_decoded =
            0;                                                          /* number of resolution decoded */
        l_img_comp->factor =
            l_cp->m_specific_param.m_dec.m_reduce; /* reducing factor per component */
        ++l_img_comp;
    }

    if (l_cp->tdx == 0 || l_cp->tdy == 0) {
        return OPJ_FALSE;
    }

    /* Compute the number of tiles */
    l_cp->tw = opj_uint_ceildiv(l_image->x1 - l_cp->tx0, l_cp->tdx);
    l_cp->th = opj_uint_ceildiv(l_image->y1 - l_cp->ty0, l_cp->tdy);

    /* Check that the number of tiles is valid */
    if (l_cp->tw == 0 || l_cp->th == 0 || l_cp->tw > 65535 / l_cp->th) {
        opj_event_msg(p_manager, EVT_ERROR,
                      "Invalid number of tiles : %u x %u (maximum fixed by jpeg2000 norm is 65535 tiles)\n",
                      l_cp->tw, l_cp->th);
        return OPJ_FALSE;
    }
    l_nb_tiles = l_cp->tw * l_cp->th;

    /* Define the tiles which will be decoded */
    if (p_j2k->m_specific_param.m_decoder.m_discard_tiles) {
        p_j2k->m_specific_param.m_decoder.m_start_tile_x =
            (p_j2k->m_specific_param.m_decoder.m_start_tile_x - l_cp->tx0) / l_cp->tdx;
        p_j2k->m_specific_param.m_decoder.m_start_tile_y =
            (p_j2k->m_specific_param.m_decoder.m_start_tile_y - l_cp->ty0) / l_cp->tdy;
        p_j2k->m_specific_param.m_decoder.m_end_tile_x = opj_uint_ceildiv(
                    p_j2k->m_specific_param.m_decoder.m_end_tile_x - l_cp->tx0,
                    l_cp->tdx);
        p_j2k->m_specific_param.m_decoder.m_end_tile_y = opj_uint_ceildiv(
                    p_j2k->m_specific_param.m_decoder.m_end_tile_y - l_cp->ty0,
                    l_cp->tdy);
    } else {
        p_j2k->m_specific_param.m_decoder.m_start_tile_x = 0;
        p_j2k->m_specific_param.m_decoder.m_start_tile_y = 0;
        p_j2k->m_specific_param.m_decoder.m_end_tile_x = l_cp->tw;
        p_j2k->m_specific_param.m_decoder.m_end_tile_y = l_cp->th;
    }

#ifdef USE_JPWL
    if (l_cp->correct) {
        /* if JPWL is on, we check whether TX errors have damaged
          too much the SIZ parameters */
        if ((l_cp->tw < 1) || (l_cp->th < 1) || (l_cp->tw > l_cp->max_tiles) ||
                (l_cp->th > l_cp->max_tiles)) {
            opj_event_msg(p_manager, JPWL_ASSUME ? EVT_WARNING : EVT_ERROR,
                          "JPWL: bad number of tiles (%d x %d)\n",
                          l_cp->tw, l_cp->th);
            if (!JPWL_ASSUME) {
                opj_event_msg(p_manager, EVT_ERROR, "JPWL: giving up\n");
                return OPJ_FALSE;
            }
            /* we try to correct */
            opj_event_msg(p_manager, EVT_WARNING, "- trying to adjust them\n");
            if (l_cp->tw < 1) {
                l_cp->tw = 1;
                opj_event_msg(p_manager, EVT_WARNING,
                              "- setting %d tiles in x => HYPOTHESIS!!!\n",
                              l_cp->tw);
            }
            if (l_cp->tw > l_cp->max_tiles) {
                l_cp->tw = 1;
                opj_event_msg(p_manager, EVT_WARNING,
                              "- too large x, increase expectance of %d\n"
                              "- setting %d tiles in x => HYPOTHESIS!!!\n",
                              l_cp->max_tiles, l_cp->tw);
            }
            if (l_cp->th < 1) {
                l_cp->th = 1;
                opj_event_msg(p_manager, EVT_WARNING,
                              "- setting %d tiles in y => HYPOTHESIS!!!\n",
                              l_cp->th);
            }
            if (l_cp->th > l_cp->max_tiles) {
                l_cp->th = 1;
                opj_event_msg(p_manager, EVT_WARNING,
                              "- too large y, increase expectance of %d to continue\n",
                              "- setting %d tiles in y => HYPOTHESIS!!!\n",
                              l_cp->max_tiles, l_cp->th);
            }
        }
    }
#endif /* USE_JPWL */

    /* memory allocations */
    l_cp->tcps = (opj_tcp_t*) opj_calloc(l_nb_tiles, sizeof(opj_tcp_t));
    if (l_cp->tcps == 00) {
        opj_event_msg(p_manager, EVT_ERROR,
                      "Not enough memory to take in charge SIZ marker\n");
        return OPJ_FALSE;
    }

#ifdef USE_JPWL
    if (l_cp->correct) {
        if (!l_cp->tcps) {
            opj_event_msg(p_manager, JPWL_ASSUME ? EVT_WARNING : EVT_ERROR,
                          "JPWL: could not alloc tcps field of cp\n");
            if (!JPWL_ASSUME) {
                opj_event_msg(p_manager, EVT_ERROR, "JPWL: giving up\n");
                return OPJ_FALSE;
            }
        }
    }
#endif /* USE_JPWL */

    p_j2k->m_specific_param.m_decoder.m_default_tcp->tccps =
        (opj_tccp_t*) opj_calloc(l_image->numcomps, sizeof(opj_tccp_t));
    if (p_j2k->m_specific_param.m_decoder.m_default_tcp->tccps  == 00) {
        opj_event_msg(p_manager, EVT_ERROR,
                      "Not enough memory to take in charge SIZ marker\n");
        return OPJ_FALSE;
    }

    p_j2k->m_specific_param.m_decoder.m_default_tcp->m_mct_records =
        (opj_mct_data_t*)opj_calloc(OPJ_J2K_MCT_DEFAULT_NB_RECORDS,
                                    sizeof(opj_mct_data_t));

    if (! p_j2k->m_specific_param.m_decoder.m_default_tcp->m_mct_records) {
        opj_event_msg(p_manager, EVT_ERROR,
                      "Not enough memory to take in charge SIZ marker\n");
        return OPJ_FALSE;
    }
    p_j2k->m_specific_param.m_decoder.m_default_tcp->m_nb_max_mct_records =
        OPJ_J2K_MCT_DEFAULT_NB_RECORDS;

    p_j2k->m_specific_param.m_decoder.m_default_tcp->m_mcc_records =
        (opj_simple_mcc_decorrelation_data_t*)
        opj_calloc(OPJ_J2K_MCC_DEFAULT_NB_RECORDS,
                   sizeof(opj_simple_mcc_decorrelation_data_t));

    if (! p_j2k->m_specific_param.m_decoder.m_default_tcp->m_mcc_records) {
        opj_event_msg(p_manager, EVT_ERROR,
                      "Not enough memory to take in charge SIZ marker\n");
        return OPJ_FALSE;
    }
    p_j2k->m_specific_param.m_decoder.m_default_tcp->m_nb_max_mcc_records =
        OPJ_J2K_MCC_DEFAULT_NB_RECORDS;

    /* set up default dc level shift */
    for (i = 0; i < l_image->numcomps; ++i) {
        if (! l_image->comps[i].sgnd) {
            p_j2k->m_specific_param.m_decoder.m_default_tcp->tccps[i].m_dc_level_shift = 1
                    << (l_image->comps[i].prec - 1);
        }
    }

    l_current_tile_param = l_cp->tcps;
    for (i = 0; i < l_nb_tiles; ++i) {
        l_current_tile_param->tccps = (opj_tccp_t*) opj_calloc(l_image->numcomps,
                                      sizeof(opj_tccp_t));
        if (l_current_tile_param->tccps == 00) {
            opj_event_msg(p_manager, EVT_ERROR,
                          "Not enough memory to take in charge SIZ marker\n");
            return OPJ_FALSE;
        }

        ++l_current_tile_param;
    }

    /*Allocate and initialize some elements of codestrem index*/
    if (!opj_j2k_allocate_tile_element_cstr_index(p_j2k)) {
        return OPJ_FALSE;
    }

    p_j2k->m_specific_param.m_decoder.m_state = J2K_STATE_MH;
    opj_image_comp_header_update(l_image, l_cp);

    return OPJ_TRUE;
}

static OPJ_BOOL opj_j2k_write_com(opj_j2k_t *p_j2k,
                                  opj_stream_private_t *p_stream,
                                  opj_event_mgr_t * p_manager
                                 )
{
    OPJ_UINT32 l_comment_size;
    OPJ_UINT32 l_total_com_size;
    const OPJ_CHAR *l_comment;
    OPJ_BYTE * l_current_ptr = 00;

    /* preconditions */
    assert(p_j2k != 00);
    assert(p_stream != 00);
    assert(p_manager != 00);

    l_comment = p_j2k->m_cp.comment;
    l_comment_size = (OPJ_UINT32)strlen(l_comment);
    l_total_com_size = l_comment_size + 6;

    if (l_total_com_size >
            p_j2k->m_specific_param.m_encoder.m_header_tile_data_size) {
        OPJ_BYTE *new_header_tile_data = (OPJ_BYTE *) opj_realloc(
                                             p_j2k->m_specific_param.m_encoder.m_header_tile_data, l_total_com_size);
        if (! new_header_tile_data) {
            opj_free(p_j2k->m_specific_param.m_encoder.m_header_tile_data);
            p_j2k->m_specific_param.m_encoder.m_header_tile_data = NULL;
            p_j2k->m_specific_param.m_encoder.m_header_tile_data_size = 0;
            opj_event_msg(p_manager, EVT_ERROR,
                          "Not enough memory to write the COM marker\n");
            return OPJ_FALSE;
        }
        p_j2k->m_specific_param.m_encoder.m_header_tile_data = new_header_tile_data;
        p_j2k->m_specific_param.m_encoder.m_header_tile_data_size = l_total_com_size;
    }

    l_current_ptr = p_j2k->m_specific_param.m_encoder.m_header_tile_data;

    opj_write_bytes(l_current_ptr, J2K_MS_COM, 2);  /* COM */
    l_current_ptr += 2;

    opj_write_bytes(l_current_ptr, l_total_com_size - 2, 2);        /* L_COM */
    l_current_ptr += 2;

    opj_write_bytes(l_current_ptr, 1,
                    2);   /* General use (IS 8859-15:1999 (Latin) values) */
    l_current_ptr += 2;

    memcpy(l_current_ptr, l_comment, l_comment_size);

    if (opj_stream_write_data(p_stream,
                              p_j2k->m_specific_param.m_encoder.m_header_tile_data, l_total_com_size,
                              p_manager) != l_total_com_size) {
        return OPJ_FALSE;
    }

    return OPJ_TRUE;
}

/**
 * Reads a COM marker (comments)
 * @param       p_j2k           the jpeg2000 file codec.
 * @param       p_header_data   the data contained in the COM box.
 * @param       p_header_size   the size of the data contained in the COM marker.
 * @param       p_manager               the user event manager.
*/
static OPJ_BOOL opj_j2k_read_com(opj_j2k_t *p_j2k,
                                 OPJ_BYTE * p_header_data,
                                 OPJ_UINT32 p_header_size,
                                 opj_event_mgr_t * p_manager
                                )
{
    /* preconditions */
    assert(p_j2k != 00)
... [truncated]

## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

