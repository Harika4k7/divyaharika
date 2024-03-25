/*#SCHOREBOARD:

  class scoreboard #(type T = AHB_Bus) extends uvm_scoreboard;
  T que[$], Err[$];
  int match,dis_match, idle;

  extern function new();
  extern virtual function void wrap();
  extern function void save_xpected();
  extern function int check_actual();
  extern function void display();
  endclass
  */

  class scoreboard extends uvm_scoreboard;
    `uvm_component_utils(scoreboard)
  uvm_tlm_analysis_fifo#(Transaction_class) mst_fifo[];

  ahb_env_cfg env_cfg;
  ahb_xtn wr_rd_xtn;

  static int pkt_rcvd,pkt_cmprd;

  covergroup cg;
  endgroup

  extern function new(string name="scoreboard",uvm_component parent);
  extern function void build_phase(uvm_phase phase);
  extern task run_phase(uvm_phase);
  extern function void report_phase(uuvm_phase phase);
  endclass

  function scoreboard::new (string name="scoreboard",uvm_component parent);
  super.new(name,parent);
  // create coverage_group here
  endfunction

  function void scoreboard::build_phase(uvm_phase phase);
  if(!uvm_config_db #(ahb_env_cfg)::get(this,"","ahb_env_cfg",env_cfg))
  `uvm_fatal("Scoreboard","cannot get env config,have you set it?");

  mst_fifo=new[env_cfg.no_of_master];
  //if we declare as dynamic for analysis_fifo use foreach loop
  foreach(mst_fifo[i])
  mst_fifo[i]=new($sformatf("mst_fifo[%0d]",i),this);

  super.build_phase(phase);
  endfunction

  task scoreboard::run_phase(uvm_phase phase);
  forever 
    begin
      mst_fifo[0].get(mst_xtn);
      pkt_rcvd++;
        if(mst_xtn.compare(slv_xtn))
        begin
          wr_xtn=mst_xtn;
          rd_xtn=mst_xtn;
          pkt_cmpr++;
          cg_w.sample();
          cg_r.sample();
          if(mst_xtn. WVALID)
            begin
              foreach(mst_xtn.WDATA[i])
              begin
                  cg_w1.sample(i);
              end
            end

            if(mst_xtn. RVALID)
            begin
              foreach(mst_xtn.RDATA[i])
              begin
                  cg_r1.sample(i);
              end
            end
          end
          else
          `uvm_error("scoreboard","Master and slave packet mismatch");
        end
    end
  endtask

  function void scoreboard:: report_phase(uvm_phase phase);
    `uvm_info("SCOREBOARD",$sformatf("No.of packets received:%0d",pkt_rcvd),UVM_LOW);
     `uvm_info("SCOREBOARD",$sformatf("No.of packets compared:%0d",pkt_cmprd),UVM_LOW);
  endfunction


  
