.. title: Converting Chemkin Format Files
.. slug: ck2cti-tutorial
.. has_math: true

.. jumbotron::

   .. raw:: html

      <h1 class="display-3">Converting Chemkin-format files</h1>

   .. class:: lead

      If you want to convert a Chemkin-format file to CTI format, or you're having
      errors when you try to do so, this section will help.

      Note that the legacy CTI input file is deprecated in Cantera 2.5, and will be
      removed in Cantera 3.0. It is superseded by the :doc:`YAML <defining-phases>`
      format. ```ck2cti``` is replaced by :doc:```ck2yaml`` <ck2yaml-tutorial>`.

CK2CTI
------

Many existing reaction mechanism files are in "CK format," by which we mean
the input file format developed for use with the Chemkin-II software package
as specified in the report describing the Chemkin software [SAND89]_.

Cantera comes with a converter utility program ``ck2cti`` (or ``ck2cti.py``)
that converts CK format into Cantera format. This program should be run from
the command line first to convert any CK files you plan to use into Cantera
format (CTI format).

Usage:

.. code:: bash

   ck2cti [--input=<filename>]
          [--thermo=<filename>]
          [--transport=<filename>]
          [--surface=<filename>]
          [--id=<phase-id>]
          [--output=<filename>]
          [--permissive]
          [-d | --debug]

Each of the terms in square brackets is an option that can be passed on the command line to
``ck2cti``.

- ``--input``: This is the chemistry input file, containing a list of all the element names that are
  used, a list of all the species names, and a list of all the reactions to be considered between
  the species. This file can also optionally contain thermodynamic information for the species.

- ``--thermo``: If the ``--input`` file does not contain the thermodynamic data, a separate file
  containing this information must be specified to the ``--thermo`` option.

- ``--transport``: Finally, the ``--input`` file can also optionally contain transport information
  for the species. If it does not, and the user wishes to use a part of Cantera that relies on some
  transport properties, the ``--transport`` option must be used to specify the file containing all
  the transport data for the species.

Example:

.. code:: bash

   ck2cti --input=chem.inp --thermo=therm.dat --transport=tran.dat

If the output file name is not given, an output file with the same name as the
input file, with the extension changed to ``.cti``.

For the case of a surface mechanism, the gas phase input file should be
specified as ``--input`` and the surface phase input file should be specified as
``--surface``.

If the ``ck2cti`` script is not on your path but the Cantera Python module is,
``ck2cti`` can also be used by running:

.. code:: bash

   python -m cantera.ck2cti --input=chem.inp --thermo=therm.dat --transport=tran.dat

An input file containing only species definitions (which can be referenced from
phase definitions in other input files) can be created by specifying only a
thermo file.

Debugging common errors in CK files
-----------------------------------

.. note::

   Many existing CK format files cause errors in ``ck2cti`` when they are
   processed. Some of these errors may be avoided by specifying the
   ``--permissive`` option. This option allows certain recoverable parsing errors
   (for example, duplicate transport or thermodynamic data) to be ignored. Other errors
   may be caused by incorrect formatting of lines in one or more of the input files.

When ``ck2cti`` encounters an error, it attempts to print the surrounding
information to help you to locate the error. Many of the most common errors
are due to an inconsistency of the input files from their standard, as defined
in the report for Chemkin referenced above. These errors include:

* Each section of the input files must be started with a keyword representing that
  section and ending with the keyword ``END``. Keywords that may begin a section
  include:

  - ``ELEMENTS`` or ``ELEM``
  - ``SPECIES`` or ``SPEC``
  - ``THERMO`` or ``THERMO ALL``
  - ``REACTIONS`` or ``REAC``
  - ``TRANSPORT``

* The thermodynamic data is read in a fixed format. This means that each
  column of the input has a particular meaning. *Many common errors are
  generated because information is missing or in the wrong column. Check
  thoroughly for extraneous or missing spaces.* The format for each
  thermodynamic entry should be as follows::

     N2                      N 2                 G200.000   6000.000  1000.00       1
      2.95258000E+00 1.39690000E-03-4.92632000E-07 7.86010000E-11-4.60755000E-15    2
     -9.23949000E+02 5.87189000E+00 3.53101000E+00-1.23661000E-04-5.02999000E-07    3
      2.43531000E-09-1.40881000E-12-1.04698000E+03 2.96747000E+00                   4

  The following table is adapted from the Chemkin manual [SAND89]_ to describe the
  column positioning of each required part of the entry. Empty columns should be
  filled with spaces.

  +---------+-------------------------------------+--------+
  |Line No. | Contents                            | Column |
  +=========+=====================================+========+
  | 1       | Species Name                        | 1–18   |
  +---------+-------------------------------------+--------+
  | 1       | Date (Optional)                     | 19–24  |
  +---------+-------------------------------------+--------+
  | 1       | Atomic Symbols and formula          | 25–44  |
  +---------+-------------------------------------+--------+
  | 1       | Phase of species (S, L, G)          | 45     |
  +---------+-------------------------------------+--------+
  | 1       | Low temperature                     | 46–55  |
  +---------+-------------------------------------+--------+
  | 1       | High temperature                    | 56–65  |
  +---------+-------------------------------------+--------+
  | 1       | Common temperature                  | 66–73  |
  +---------+-------------------------------------+--------+
  | 1       | Additional Atomic Symbols           | 74–78  |
  +---------+-------------------------------------+--------+
  | 1       | The integer ``1``                   | 80     |
  +---------+-------------------------------------+--------+
  | 2       | Coefficients :math:`a_1`            | 1–75   |
  |         | to :math:`a_5` for the upper        |        |
  |         | temperature interval                |        |
  +---------+-------------------------------------+--------+
  | 2       | The integer ``2``                   | 80     |
  +---------+-------------------------------------+--------+
  | 3       | Coefficients :math:`a_6,\ a_7`      | 1–75   |
  |         | for the upper temperature interval, |        |
  |         | and :math:`a_1,\ a_2,\ a_3` for     |        |
  |         | the lower temperature interval      |        |
  +---------+-------------------------------------+--------+
  | 3       | The integer ``3``                   | 80     |
  +---------+-------------------------------------+--------+
  | 4       | Coefficients :math:`a_4` through    | 1–60   |
  |         | :math:`a_7` for the lower           |        |
  |         | temperature interval                |        |
  +---------+-------------------------------------+--------+
  | 4       | The integer ``4``                   | 80     |
  +---------+-------------------------------------+--------+

  The first 18 columns are reserved for the species name. The name assigned
  to the species in the thermodynamic data must be the same as the species
  name defined in the ``SPECIES`` section. If the species name is shorter
  than 18 characters, the rest of the characters should be filled by spaces.
  The next six columns (columns 19–24) are typically used to write a date;
  they are not used further. The next 20 columns (25–44) are used to
  specify the elemental composition of the species. In column 45, the phase
  of the species (``S``, ``L``, or ``G`` for solid, liquid, or gas
  respectively) should be specified. The next 28 columns are reserved for
  the temperatures that delimit the ranges of the polynomials specified on
  the next several lines. The first two temperatures have a width of 10
  columns each (46–55 and 56–65), and represent the lowest temperature and
  highest temperature for which the polynomials are valid. The last
  temperature has a width of 8 columns (66–73) and is the "common"
  temperature, where the switch from low to high occurs. The next 5 columns
  (74–78) are reserved for atomic symbols and are usually left blank for
  the default behavior. Column 79 is blank and finally, the row is ended in
  column 80 with the integer ``1``.

  The next three lines of the thermodynamic entry have a similar format.
  They contain the coefficients of the polynomial described in
  :ref:`sec-thermo-models` for the NASA 7-coefficient polynomial formulation.
  The second row of the thermo entry (the first after the information row)
  contains the first five coefficients that apply the the temperature range
  between the midpoint and the upper limit. 15 columns are alloted for each
  coefficient (for a total of 75 columns), with no spaces between them.
  Although the entry above shows spaces between positive coefficients, it is
  to be noted that this is done only for formatting consistency with other
  lines that contain negative numbers. After the coefficients, four spaces
  in columns 76–79 are followed by the integer ``2`` in column 80. On the
  next line, the last two coefficients for the upper temperature range and
  the first three coefficients for the lower temperature range are
  specified. Once again, this takes up the first 75 columns, columns 76–79
  are blank, and the integer ``3`` is in column 80. Finally, on the last
  line of a particular entry, the last four coefficients of the lower
  temperature range are specified in columns 1–60, 19 blank spaces are
  present, and the integer ``4`` is in column 80. The 19 blank spaces in the
  last line are part of the standard. However, since the original Chemkin
  interpreter ignored those spaces, researchers began using that space to
  store additional information that was not necessary for the input file.
  Although these numbers create an error in ``ck2cti`` if present, they are
  harmless and can be ignored by using the ``--permissive`` option.

  If the number of atoms of an element in a thermodynamic entry has more than 3
  digits, it will cause a conversion error. To avoid the error, the element
  symbol should have a ``0`` in the first line of the entry. An ampersand
  (``&``) is added after the index of the first line, and the element symbols
  and their amounts should be written on the next line as follows::

     BIN6J      PYRENEJ1     C   0H   0    0    0G   300.000  5000.000 1401.000     1&
     C 778    H 263
      3.63345177E+01 3.13968020E-02-1.09044660E-05 1.71125597E-09-1.00056355E-13    2
      4.05143093E+04-1.77494305E+02-1.20603441E+01 1.59247554E-01-1.41562602E-04    3
      6.26071650E-08-1.09305161E-11 5.56473533E+04 7.68451211E+01                   4

  or on separate lines with ampersand (``&``) as the last character on the line::

     BIN6       PYRENE       C   0H   0    0    0G   300.000  5000.000 1401.000     1&
     C      778&
     H      264
      3.65839677E+01 3.36764102E-02-1.16783938E-05 1.83077466E-09-1.06963777E-13    2
      9.29809483E+03-1.81272070E+02-1.29758980E+01 1.63790064E-01-1.43851166E-04    3
      6.31057915E-08-1.09568047E-11 2.48866399E+04 7.94950474E+01                   4

* It may be the case that scientific formatted numbers are missing the ``E``.
  In this case, numbers often show up as ``1.1+01``, when they should be
  ``1.1E+01``. You can fix this with a simple Regular Expression find and
  replace::

     Find: (\d+\.\d+)([+-]\d+)
     Replace: \1E\2

* The transport data file also has a specified format, as described in
  [SAND98]_, although the format is not as strict as for the thermodynamic
  entries. In particular, the first 15 columns of a line are reserved for
  the species name. *One common source of errors is a species that is present
  in the transport data file, but not in the thermodynamic data or in
  the species list; or a species that is present in the species list but
  not the transport data file.* The rest of the columns on a given line have
  no particular format, but must be present in the following order:

  +------------------+------------------------------------------------------------+
  | Parameter Number | Parameter Name                                             |
  +==================+============================================================+
  | 1                | An integer with value 0, 1, or 2 indicating                |
  |                  | monatomic, linear, or non-linear molecular geometry.       |
  +------------------+------------------------------------------------------------+
  | 2                | The Lennard-Jones potential well depth                     |
  |                  | :math:`\varepsilon/k_B` in Kelvin                          |
  +------------------+------------------------------------------------------------+
  | 3                | The Lennard-Jones collision diameter :math:`\sigma`        |
  |                  | in Angstrom                                                |
  +------------------+------------------------------------------------------------+
  | 4                | The dipole moment :math:`\mu` in Debye                     |
  +------------------+------------------------------------------------------------+
  | 5                | The polarizability :math:`\alpha` in Angstroms cubed       |
  +------------------+------------------------------------------------------------+
  | 6                | The rotational relaxation collision number                 |
  |                  | :math:`Z_{rot}` at 298 K                                   |
  +------------------+------------------------------------------------------------+

  Another common error is if all 6 of these numbers are not present for every
  species.

.. [SAND89] See R. J. Kee, F. M. Rupley, and J. A. Miller, Sandia National
   Laboratories Report SAND89-8009 (1989).
   http://www.osti.gov/scitech/biblio/5681118

.. [SAND98] See R. J. Kee, G. Dixon-Lewis, J. Warnatz, M. E. Coltrin, J. A. Miller,
   H. K. Moffat, Sandia National Laboratories Report SAND86-8246B (1998).
