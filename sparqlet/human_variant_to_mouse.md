# Convert human vriants to mouse genome positions

* Search human variants with VEP annotation and Clinver significance (TogoVar RDF)
* Human Ensembl transcript to mouse Ensembl transcript via Homologene (Homologene RDF)
  * Convert Ensembl transcript from/to NCBI gene (TogoID RDF)
* Alaign by ggsearch (global alignment) (local API)
  * Get CDS (Ensembl API)
* Calculate mouse genome (GRCm39/mm39) position
  * Get exon-intron structure and translation info (Ensembl API)
* Get mouse strain varants (GRCm38/mm10) (MoG+ API) and compare
  * Convert from/to GRCm39/mm39 (UCSC API)

## Parameters

* `hgnc` : id or symbol
  * default: ABCA12
  * example: 14637
* `clinvar` : filter by Clinvar
  * default: true
* `strain_match` : human-alt mouse-strain-ref match
  * default: true

## `id`
```javascript
({hgnc, clinvar})=>{
  if (clinvar.match(/^false$/)) clinvar = false;
  if (hgnc.match(/[A-Z]/)) return {symbol: hgnc, clinvar: clinvar};
  else return {hgnc: hgnc, clinvar: clinvar};
}
```

## Endpoint

https://grch38.togovar.org/sparql

## `var_info`
```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX tgvo: <http://togovar.biosciencedbc.jp/vocabulary/>
PREFIX cvo: <http://purl.jp/bio/10/clinvar/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX gvo: <http://genome-variation.org/resource#>
PREFIX hgnc: <http://identifiers.org/hgnc/>
SELECT DISTINCT ?cons ?hgvsc ?hgvsp ?togovar ?hgnc (GROUP_CONCAT(DISTINCT ?interprt ; separator = ", " ) AS ?interpretation )
FROM <http://togovar.biosciencedbc.jp/so>
FROM <http://togovar.biosciencedbc.jp/hgnc>
FROM <http://togovar.biosciencedbc.jp/variant/annotation/ensembl>
{{#if id.clinvar}}
FROM <http://togovar.biosciencedbc.jp/clinvar>
FROM <http://togovar.biosciencedbc.jp/variant/annotation/clinvar>
{{/if}}
WHERE {
  {{#if id.symbol}}
  GRAPH <http://togovar.biosciencedbc.jp/hgnc> {
    ?hgnc rdfs:label "{{id.symbol}}" .
  }
  {{/if}}
  GRAPH <http://togovar.biosciencedbc.jp/variant/annotation/ensembl> { 
    ?togovar tgvo:hasConsequence ?cons_node .
    ?cons_node a ?cons_type ;
               tgvo:hgvsc ?hgvsc ;
               tgvo:hgnc {{#if id.symbol}}?hgnc{{/if}}{{#if id.hgnc}}hgnc:{{id.hgnc}}{{/if}} .
    OPTIONAL {
      ?cons_node tgvo:hgvsp ?hgvsp .
    }
    FILTER (REGEX (?hgvsc, "ENST"))
  }
  GRAPH <http://togovar.biosciencedbc.jp/so> {
    ?cons_type rdfs:label ?cons .
  }
{{#if id.clinvar}}
  GRAPH <http://togovar.biosciencedbc.jp/variant/annotation/clinvar> {
    ?togovar dct:identifier ?var_id .
    BIND(IRI(CONCAT("http://ncbi.nlm.nih.gov/clinvar/variation/", ?var_id)) AS ?clinvar)
  }
  GRAPH <http://togovar.biosciencedbc.jp/clinvar> {
    ?clinvar cvo:interpreted_record/cvo:rcv_list/cvo:rcv_accession/cvo:interpretation ?interprt .
  }
{{/if}}
}
```

## `ensts`
```javascript
({var_info})=>{
  function uniq(array) {
    return Array.from(new Set(array));
  }
  return "enst:" + uniq(var_info.results.bindings.map(d=>d.hgvsc.value.split(/\./)[0])).join(" enst:");
}
```

## Endpoint

http://ep.dbcls.jp/togoid/sparql

## `human_ncbigene`
```sparql
PREFIX enst: <http://identifiers.org/ensembl/>
PREFIX ncbigene: <http://identifiers.org/ncbigene/>
SELECT DISTINCT ?id ?enst
WHERE {
  VALUES ?enst { {{ensts}} }
  {
    GRAPH <http://togoid.dbcls.jp/graph/ensembl_transcript-ncbigene> {
      ?enst [] ?id .
    }
  } UNION {
    GRAPH <http://togoid.dbcls.jp/graph/ncbigene-ensembl_transcript> {
      ?id [] ?enst .
    }  
  }
}
```

## Endpoint

https://togodx-dev.dbcls.jp/human/sparql

## `mouse_ncbigene`
```sparql
PREFIX orth: <http://purl.org/net/orth#>
PREFIX homologene: <https://ncbi.nlm.nih.gov/homologene/>
PREFIX taxid: <http://identifiers.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT ?id
FROM <http://rdf.integbio.jp/dataset/togosite/homologene/ontology>
FROM <http://rdf.integbio.jp/dataset/togosite/homologene/data>
WHERE {
  [] orth:inDataset homologene: ;
     orth:hasHomologousMember <{{human_ncbigene.results.bindings.0.id.value}}> ;
     orth:hasHomologousMember ?id .
  ?id orth:taxon taxid:10090 .
}
```

## Endpoint

http://ep.dbcls.jp/togoid/sparql

## `mouse_enst_pre`
```sparql
PREFIX enst: <http://identifiers.org/ensembl/>
PREFIX ncbigene: <http://identifiers.org/ncbigene/>
SELECT DISTINCT ?id
WHERE {
  {
    GRAPH <http://togoid.dbcls.jp/graph/ensembl_transcript-ncbigene> {
      ?id [] <{{mouse_ncbigene.results.bindings.0.id.value}}> .
    }
  } UNION {
    GRAPH <http://togoid.dbcls.jp/graph/ncbigene-ensembl_transcript> {
      <{{mouse_ncbigene.results.bindings.0.id.value}}> [] ?id .
    }  
  }
}
```

## `sequence`
```javascript
async ({strain_match, var_info, human_ncbigene, mouse_enst_pre})=>{
  if (strain_match.match(/^false$/)) strain_match = false;
  const human_enst = human_ncbigene.results.bindings[0].enst.value.split(/\//)[4];
  const mouse_enst = mouse_enst_pre.results.bindings[0].id.value.split(/\//)[4];
  const api = "https://rest.ensembl.org";
  const hsa = await fetch(api + "/sequence/id/" + human_enst + "?type=cds&content-type=text/x-fasta").then(d=>d.text());
  const mmu = await fetch(api + "/sequence/id/" + mouse_enst + "?type=cds&content-type=text/x-fasta").then(d=>d.text());
  console.log(hsa);
  console.log(mmu);
  let options = {
    method: 'POST',
    body: 'hsa=' + hsa + '&mmu=' + mmu,
    headers: {
      'Accept': 'text/plain',
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  };

  const res = await fetch("https://sparql-support.dbcls.jp/api/ggsearch", options).then(d=>d.text());
  console.log(res);
  const lines = res.split(/\n/);
  const hsa_info = await fetch(api + "/lookup/id/" + human_enst + "?expand=1&content-type=application/json").then(d=>d.json());
  const mmu_info = await fetch(api + "/lookup/id/" + mouse_enst + "?expand=1&content-type=application/json").then(d=>d.json());
  console.log(hsa_info);
  console.log(mmu_info);

  let mmu_ref = {};
  let mmu_var = {};
  if (strain_match) {
    ////// GRCm39/mm39 > GRCm38/mm10 LiftOver
    let lift_url = "https://genome.ucsc.edu/cgi-bin/hgLiftOver";
    let lift_body = "hglft_fromDb=mm39&hglft_toDb=mm10&hglft_minMatch=0.95&hglft_userData=chr" + mmu_info.seq_region_name + "%20" + mmu_info.start + "%20" + mmu_info.end;
    let lift_html = await fetch(lift_url, {method: "POST", body: lift_body}).then(d=>d.text());
    let lift_file = "https://genome.ucsc.edu/trash/" + lift_html.match(/<A HREF=..\/trash\/(.+.bed) TARGET=_blank>View Conversions<\/A>/)[1];
    let lift_res = await fetch(lift_file).then(d=>d.text());
    let lift_array = lift_res.split(/\s+/);
    lift_array[0] = lift_array[0].replace(/chr/, '');
    //////
    
    //options.body += "chrName=" + mmu_info.seq_region_name + "&chrStart=" + mmu_info.start + "&chrEnd=" + mmu_info.end + "&strainNoSlct=refGenome&strainNoSlct=msmv4_sq&strainNoSlct=jf1v3&strainNoSlct=kjrv1&strainNoSlct=swnv1&strainNoSlct=chdv1&strainNoSlct=njlv1&strainNoSlct=blg2v1&strainNoSlct=hmiv1&strainNoSlct=bfmv1&strainNoSlct=pgn2v1&strainNoSlct=129P2_OlaHsd&strainNoSlct=129S1_SvImJ&strainNoSlct=129S5SvEvBrd&strainNoSlct=A_J&strainNoSlct=AKR_J&strainNoSlct=BALB_cJ&strainNoSlct=BTBR_T%2B_Itpr3tf_J&strainNoSlct=BUB_BnJ&strainNoSlct=C3H_HeH&strainNoSlct=C3H_HeJ&strainNoSlct=C57BL_10J&strainNoSlct=C57BL_6NJ&strainNoSlct=C57BR_cdJ&strainNoSlct=C57L_J&strainNoSlct=C58_J&strainNoSlct=CAST_EiJ&strainNoSlct=CBA_J&strainNoSlct=DBA_1J&strainNoSlct=DBA_2J&strainNoSlct=FVB_NJ&strainNoSlct=I_LnJ&strainNoSlct=KK_HiJ&strainNoSlct=LEWES_EiJ&strainNoSlct=LP_J&strainNoSlct=MOLF_EiJ&strainNoSlct=NOD_ShiLtJ&strainNoSlct=NZB_B1NJ&strainNoSlct=NZO_HlLtJ&strainNoSlct=NZW_LacJ&strainNoSlct=PWK_PhJ&strainNoSlct=RF_J&strainNoSlct=SEA_GnJ&strainNoSlct=SPRET_EiJ&strainNoSlct=ST_bJ&strainNoSlct=WSB_EiJ&strainNoSlct=ZALENDE_EiJ&seqType=genome&chrName=5&geneNameSearchText=&index=submit&presentType=dwnld"
    options.body += "chrName=" + lift_array[0] + "&chrStart=" + parseInt(lift_array[1]) + "&chrEnd=" + parseInt(lift_array[2]) + "&strainNoSlct=refGenome&strainNoSlct=msmv4_sq&strainNoSlct=jf1v3&strainNoSlct=kjrv1&strainNoSlct=swnv1&strainNoSlct=chdv1&strainNoSlct=njlv1&strainNoSlct=blg2v1&strainNoSlct=hmiv1&strainNoSlct=bfmv1&strainNoSlct=pgn2v1&strainNoSlct=129P2_OlaHsd&strainNoSlct=129S1_SvImJ&strainNoSlct=129S5SvEvBrd&strainNoSlct=A_J&strainNoSlct=AKR_J&strainNoSlct=BALB_cJ&strainNoSlct=BTBR_T%2B_Itpr3tf_J&strainNoSlct=BUB_BnJ&strainNoSlct=C3H_HeH&strainNoSlct=C3H_HeJ&strainNoSlct=C57BL_10J&strainNoSlct=C57BL_6NJ&strainNoSlct=C57BR_cdJ&strainNoSlct=C57L_J&strainNoSlct=C58_J&strainNoSlct=CAST_EiJ&strainNoSlct=CBA_J&strainNoSlct=DBA_1J&strainNoSlct=DBA_2J&strainNoSlct=FVB_NJ&strainNoSlct=I_LnJ&strainNoSlct=KK_HiJ&strainNoSlct=LEWES_EiJ&strainNoSlct=LP_J&strainNoSlct=MOLF_EiJ&strainNoSlct=NOD_ShiLtJ&strainNoSlct=NZB_B1NJ&strainNoSlct=NZO_HlLtJ&strainNoSlct=NZW_LacJ&strainNoSlct=PWK_PhJ&strainNoSlct=RF_J&strainNoSlct=SEA_GnJ&strainNoSlct=SPRET_EiJ&strainNoSlct=ST_bJ&strainNoSlct=WSB_EiJ&strainNoSlct=ZALENDE_EiJ&seqType=genome&chrName=5&geneNameSearchText=&index=submit&presentType=dwnld"
    const mogp = await fetch("https://molossinus.brc.riken.jp/mogplus/variantTable/", options).then(d=>d.text());
    console.log(mogp);
    const mogp_r = mogp.split(/\n/);
    let pos_list = mogp_r[0].split(/\t/);
    if (!pos_list[1]) return [];

    ////// GRCm38/mm10 > GRCm39/mm39 LiftOver 
    lift_body = "hglft_fromDb=mm10&hglft_toDb=mm39&hglft_minMatch=0.95&hglft_userData=";
    for (let i = 1; i < pos_list.length; i++){
      lift_body += "chr" + lift_array[0] + "%20" + pos_list[i] + "%20" + pos_list[i] + "%0A";
    }
    lift_html = await fetch(lift_url, {method: "POST", body: lift_body}).then(d=>d.text());
    lift_file = "https://genome.ucsc.edu/trash/" + lift_html.match(/<A HREF=..\/trash\/(.+.bed) TARGET=_blank>View Conversions<\/A>/)[1];
    lift_res = await fetch(lift_file).then(d=>d.text());
    console.log(lift_res);
    lift_array = lift_res.split(/\n/);
    for (let i = 0; i < lift_array.length; i++) {
      let tmp = lift_array[i].split(/\s+/);
      pos_list[i+1] = tmp[1]; 
    }
    //////

    const ref_list = mogp_r[1].split(/\t/);
    for (let i = 2; i < mogp_r.length; i++) {
      const alt = mogp_r[i].split(/\t/);
      for (let j = 1; j < alt.length; j++) {
        if (!alt[j].match(/[A-Z]/)) continue;
        const vs = pos_list[j] + ref_list[j] + ">" + alt[j];
        console.log(vs);
        if (!mmu_var[vs]) mmu_var[vs] = [];
        mmu_var[vs].push(alt[0]);
      }
    }
  }
  
  let result = [];
  const re = new RegExp(human_enst);
  for (let i = 0; i < var_info.results.bindings.length; i++) {
    if (var_info.results.bindings[i].hgvsc.value.match(re)) {
      let pick_info = var_info.results.bindings[i];

      let pos = pick_info.hgvsc.value.match(/:c\.([-\*]*\d+)/)[1];
      let vep_delta = 0;
      if (pick_info.hgvsc.value.match(/:c\.\d+[\+-]\d+/)) vep_delta = parseInt(pick_info.hgvsc.value.match(/:c\.\d+([\+-]\d+)/)[1]);

  let hsa_cds_pos = 0;
  let mmu_cds_pos = 0;
  let ref_line_pos = 0;
  let alt_line_pos = 0;
  let r_nuc;
  let a_nuc;
  let hgvsp = pick_info.hgvsp ?? false;
  hgvsp = hgvsp.value ?? false;
  let interpretation = pick_info.interpretation ?? false;
  interpretation = interpretation.value ?? false;
      
  if (pos.match(/-/) || pos.match(/\*/)) {
    vep_delta = parseInt(pos.replace(/\*/, ""));
    if (pos < 0) pos = 1;
    else pos = (hsa_info.Translation.length + 1) * 3;
  }
  pos = parseInt(pos);
  for (let l of lines) {
    if (l.match(/global\/global \(N-W\).+overlap \(\d+-\d+:\d+-\d+\)/)) {
      let tmp = l.match(/global\/global \(N-W\).+overlap \((\d+)-\d+:(\d+)-\d+\)/);
      hsa_cds_pos = tmp[1] - 1;
      mmu_cds_pos = tmp[2] - 1;	
    }
    if (l.match(/^ENST/)) {
      let seq = l.match(/^ENST.+ +(.+)/)[1];
      if (hsa_cds_pos + seq.replace(/-/g, "").length < pos) {
        hsa_cds_pos += seq.replace(/-/g, "").length;
      } else {
        let nucs = seq.split('');
        for (let n of nucs) {
          ref_line_pos++;
          if (n.match(/[ATGCN]/)) hsa_cds_pos++;
          if (hsa_cds_pos == pos) {
            r_nuc = n;
            if (vep_delta != 0) r_nuc = "-";
            break;
          }
        }
      } 
    }
    if (l.match(/^ENSMU/)) {
      let seq = l.match(/^ENSMU.+ +(.+)/)[1];
      if (ref_line_pos == 0) {
        mmu_cds_pos += seq.replace(/-/g, "").length;
      } else {
        let nucs = seq.split('');
        for (let n of nucs) {
          alt_line_pos++;
          if (n.match(/[ATGCN]/)) mmu_cds_pos++;
          if (alt_line_pos == ref_line_pos) {
            a_nuc = n;
            if (vep_delta != 0) a_nuc = "-";
            break;
          }
        }
      } 
    }
    if (alt_line_pos && ref_line_pos == alt_line_pos) break;
  }

  let hsa_pos = 0;
  let cds_pos_h = 0;
  if (hsa_info.strand == 1) {
    hsa_pos = hsa_info.Translation.start - 1;
    for (let i = 0; i < hsa_info.Exon.length; i++) {
      const exon = hsa_info.Exon[i];
      if (exon.end < hsa_pos) continue;
      if (exon.end - hsa_pos < hsa_cds_pos - cds_pos_h) {
        cds_pos_h += exon.end - hsa_pos;
        hsa_pos = hsa_info.Exon[i+1].start - 1;
      } else {
        hsa_pos += hsa_cds_pos - cds_pos_h;
        break;
      }
    }
  } else {
    hsa_pos = hsa_info.Translation.end + 1;
    for (let i = 0; i < hsa_info.Exon.length; i++) {
      const exon = hsa_info.Exon[i];
      if (hsa_pos < exon.start) continue;
      if (hsa_pos - exon.start < hsa_cds_pos - cds_pos_h) {
        cds_pos_h += hsa_pos - exon.start;
        hsa_pos = hsa_info.Exon[i+1].end + 1;
      } else {
        hsa_pos -= hsa_cds_pos - cds_pos_h;
        break;
      }
    }
  }

  let mmu_pos = 0;
  let cds_pos_m = 0;
  if (mmu_info.strand == 1) {
    mmu_pos = mmu_info.Translation.start - 1;
    for (let i = 0; i < mmu_info.Exon.length; i++) {
      const exon = mmu_info.Exon[i];
      if (exon.end < mmu_pos) continue;
      if (exon.end - mmu_pos < mmu_cds_pos - cds_pos_m) {
        cds_pos_m += exon.end - mmu_pos;
        mmu_pos = mmu_info.Exon[i+1].start - 1;
      } else {
        mmu_pos += mmu_cds_pos - cds_pos_m;
        break;
      }
    }
  } else {
    mmu_pos = mmu_info.Translation.end + 1;
    for (let i = 0; i < mmu_info.Exon.length; i++) {
      const exon = mmu_info.Exon[i];
      if (mmu_pos < exon.start) continue;
      if (mmu_pos - exon.start < mmu_cds_pos - cds_pos_m) {
        cds_pos_m += mmu_pos - exon.start;
        mmu_pos = mmu_info.Exon[i+1].end + 1;
      } else {
        mmu_pos -= mmu_cds_pos - cds_pos_m;
        break;
      }
    }
  }

   mmu_pos += vep_delta * mmu_info.strand;
   let vs = false
   if (pick_info.hgvsc.value.match(/[A-Z]>[A-Z]/)) vs = mmu_pos + pick_info.hgvsc.value.match(/([A-Z]>[A-Z])/)[1];
   if (!strain_match || (strain_match && mmu_var[vs])) {
    result.push({
    uri: pick_info.togovar.value,
    hgvsc: pick_info.hgvsc.value,
    hgvsp: hgvsp,
    consequence: pick_info.cons.value,
    significance: interpretation,
    human: {
      assembly: hsa_info.assembly_name,
      chromosome: hsa_info.seq_region_name,
      position: hsa_pos + (vep_delta * hsa_info.strand),
      strand: hsa_info.strand,
      ref: r_nuc
    },
    mouse: {
      assembly: mmu_info.assembly_name,
      chromosome: mmu_info.seq_region_name,
      position: mmu_pos,
      srrand: mmu_info.strand,
      ref: a_nuc,
      strain: mmu_var[vs] || false
    }
  });
  }

    }
  }
  return result;
}
```
