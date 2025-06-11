public byte[] Serialize()
        {
            var image = BuildImage();

            using (var ms = new MemoryStream())
            {
                foreach (var section in image)
                {
                    var align = section.Align;
                    align = 4; // force natural alignment
                    if (align > 1)
                    {
                        var rem = ms.Length % align;
                        if (rem > 0)
                        {
                            while (rem++ < align)
                                ms.WriteByte(0);
                        }
                    }
                    var raw = section.GetRaw();
                    section.BaseAddress = ms.Length;
                    section.Length = raw.Length;
                    ms.Write(raw, 0, raw.Length);
                }

                foreach (var section in image)
                {
                    foreach (var r in section.Relocations)
                    {
                        if (r is Stub_6_Relocation s6r)
                        {
                            var shdr = (IMG_SectionHeader)image.First(x => x.Name == IMG_SectionHeader.sName);

                            var r_sec = (IMG_DataTable<__PILOT_IMG_HEADER>)section; // image.First(x => x.Name == r.DefinedIn)
                            var r_offset = s6r.Offset * Marshal.SizeOf(typeof(__PILOT_IMG_HEADER));
                            var Stub_6_offset = (int)Marshal.OffsetOf(typeof(__PILOT_IMG_HEADER), nameof(__PILOT_IMG_HEADER.Stub_6));
                            var r_size = sizeof(IEC61131_DWORD);
                            var r_value = BitConverter.GetBytes((IEC61131_DWORD)shdr.BaseAddress);

                            // patch it
                            ms.Seek(r_sec.BaseAddress + r_offset + Stub_6_offset, SeekOrigin.Begin);
                            ms.Write(r_value, 0, r_size);
                        }
                        else
                        {
                            switch (r.Kind)
                            {
                                case IMG_RelocationKind.R_Find_Symbol:
                                {
                                    var shStrTab = (IMG_StringTable)image.First(x => x.Name == ".shstrtab");

                                    var r_sec = section;
                                    var r_offset = r.Offset;
                                    var r_size = r.Length;
                                    var r_value = BitConverter.GetBytes((IEC61131_DWORD)shStrTab.OffsetOf(r.Symbol));

                                    // patch it
                                    ms.Seek(r_sec.BaseAddress + r_offset, SeekOrigin.Begin);
                                    ms.Write(r_value, 0, r_size);

                                    break;
                                }
                                case IMG_RelocationKind.R_Section_RVA:
                                {
                                    var shdr = (IMG_SectionHeader)section;

                                    var r_sec = image.First(x => x.Name == r.Symbol);
                                    var r_offset = r.Offset;
                                    var r_size = r.Length;
                                    var r_value = BitConverter.GetBytes((IEC61131_DWORD)r_sec.BaseAddress);

                                    // patch it
                                    ms.Seek(shdr.BaseAddress + r_offset, SeekOrigin.Begin);
                                    ms.Write(r_value, 0, r_size);
                                    break;
                                }
                                case IMG_RelocationKind.R_Section_Size:
                                {
                                    var shdr = (IMG_SectionHeader)section;

                                    var r_sec = image.First(x => x.Name == r.Symbol);
                                    var r_offset = r.Offset;
                                    var r_size = r.Length;
                                    var r_value = BitConverter.GetBytes((IEC61131_DWORD)r_sec.Length);

                                    // patch it
                                    ms.Seek(shdr.BaseAddress + r_offset, SeekOrigin.Begin);
                                    ms.Write(r_value, 0, r_size);
                                    break;
                                }
                                default:
                                    break;
                            }
                        }
                    }
                }

                return ms.ToArray();
            }
        }
