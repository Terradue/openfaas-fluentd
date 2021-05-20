FROM openfaas/of-watchdog:0.7.2 as watchdog
FROM ubuntu:20.04

COPY --from=watchdog /fwatchdog /usr/bin/fwatchdog

RUN chmod +x /usr/bin/fwatchdog                                                     && \
    groupadd --gid 1002 app                                                         && \
    useradd --comment "Default user"                                                   \
    --create-home                                                                      \
    --gid 1002                                                                         \
    --no-log-init                                                                      \
    --shell /bin/bash                                                                  \
    --uid 1002 app                                                                  && \
    chown app /home/app

ENV PATH=$PATH:/home/app/.local/bin:/srv/conda/envs/env_openfaas/bin                    \
    MAMBA_ROOT_PREFIX=/srv/conda                                                        \
    fprocess="td-agent -c /etc/td-agent/td-agent.conf"                                  \
    mode="http"                                                                         \
    upstream_url="http://127.0.0.1:8000"    \
    http_port=8000                                               

RUN apt-get update                                                                                                          && \
    apt-get install -y ca-certificates wget bash bzip2 curl gnupg2                                                          && \                                                               
    curl https://packages.treasuredata.com/GPG-KEY-td-agent | apt-key add -                                                 && \
    echo "deb http://packages.treasuredata.com/4/ubuntu/focal/ focal contrib" > /etc/apt/sources.list.d/treasure-data.list  && \
    apt-get update                                                                                                          && \
    apt-get install -y td-agent                                                                                             && \
    mkdir -p /etc/td-agent                                                                                                  && \
    touch /var/log/json.log  && \
    mkdir -p /var/log/td-agent/ && \
    chown -R 1002:1002 /var/log/td-agent/
    

COPY td-agent.conf /etc/td-agent/td-agent.conf

USER app

HEALTHCHECK --interval=5s CMD [ -e /tmp/.lock ] || exit 1

WORKDIR /home/app/

CMD ["fwatchdog"]