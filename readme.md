#Library of functions to interact with fasta files</br>
</br>
This library provides functions to facilitate interaction with `fasta` files. It includes functions to read/write a fasta file, rename fasta sequences from alphabetical names to numeric and vice versa. The read function also can work with inputing a char* of the fasta sequences rather than the file. This is because it was originally written to work as part of a mapper function for MRMPI library. However, now it works as a standalone function and the MRMPI library is not required to use FastaToolbox library.</br>
</br>
**read_fasta:**</br>
This function works in two modes:
```c++
char* filename = "file.fasta";
FastaToolbox fasta_toolbox;
std::list<Protein> proteins = fasta_toolbox.read_fasta(filename);
```
It receives a char array (char*) of the name of the file and returns a list of `Protein`s. Each `Protein` is an instance of the struct `Protein`, that originally has a std::string of protein/DNA name and a std::string of protein/DNA sequence.

The `read_fasta()` function also can work with inputing a char* of the fasta sequences rather than the filename. This is because it was originally written to work as part of a mapper function for MRMPI library. However, now it works as a standalone function and the MRMPI library is not required to use FastaToolbox library. If the user needs to use this function with the MRMPI library, they will need to write a mapper function as below:
```c++
void read_proteins_mapper(int itask, char* text, int size, KeyValue* kv, void *local_seqs_ptr) {
	FastaToolbox fasta_toolbox;
	std::list<Protein> proteins = fasta_toolbox.read_proteins(text, size, itask);
	for (auto iter = proteins.begin(); iter != proteins.end(); iter++) {
		Protein protein = *iter;
		kv -> add(&protein.name[0], protein.name.length() + 1, &protein.sequence[0], protein.sequence.length() + 1);
	}
}
```
For this call, the MRMPI library breaks down the file to multiple parts and passing each section to a mapper. The `read_fasta` then accepts only the section of the file designated to that mapper, and extracts the text in that section into a list of `Protein`s.<\br>
</br>
**convert_name_to_num:**</br>
This function assigns numerical IDs to the sequences in the Proteins list of the instance. If the function read_conversion_table is called before convert_name_to_num, it will relate the conversion table that was stored on a file to the sequences in the list, assigning new IDs only to the sequences that were not present in the conversion table.</br>
The second argument of the function is optional. It is for the files where a sequence is broken down to multiple parts, each part staored as a different sequence with a "_sectionID" suffix. This is appropriated for my use in the NADDA project, where I was detecting conserved regions within a sequence and then processing these regions separately.</br>
**convert_num_to_name:**</br>
Similar to `convert_num_to_name`, but changing from numbered sequences to names based on the conversion table. Note that a call to read_conversion_table or convert_name_to_num is required before calling this function. This is because the former two functions are the only functions that will assign a numerical ID to names.</br>
</br>
**write_fasta:**</br>
**write_conversion_table:**</br>
Both of these functions receive a single string argument that describes the filename where the fasta file or the converion table from sequence names to sequence ID numbers should be written.</br>
</br>
**convert_pfam_file_to_numbered:**</br>
This function also requires a previous call to read_conversion_table to convert_name_to_num, so that numerical IDs will be assigned to the sequences while calling the function. This function opens the file specificed by the first argument and is a result of running pfam on a fasta file(with output option csv/tsv), and prints a copy of this file on a file named by the sequence argument, replacing the sequence names by their corresponding numerical IDs as they appear in the conversion table.</br>

