<script lang="ts" setup>
import { postLabels } from "@/constants/labels";
import { formatDatetime } from "@/utils/date";
import type { ListedPost } from "@halo-dev/api-client";
import { consoleApiClient } from "@halo-dev/api-client";
import {
  IconExternalLinkLine,
  VCard,
  VEntity,
  VEntityContainer,
  VEntityField,
  VSpace,
} from "@halo-dev/components";
import { useQuery } from "@tanstack/vue-query";
import { OverlayScrollbarsComponent } from "overlayscrollbars-vue";

const { data } = useQuery<ListedPost[]>({
  queryKey: ["widget-recent-posts"],
  queryFn: async () => {
    const { data } = await consoleApiClient.content.post.listPosts({
      labelSelector: [
        `${postLabels.DELETED}=false`,
        `${postLabels.PUBLISHED}=true`,
      ],
      sort: ["spec.publishTime,desc"],
      page: 1,
      size: 10,
    });
    return data.items;
  },
});
</script>
<template>
  <VCard
    :body-class="['h-full', '!p-0', '!overflow-auto']"
    class="h-full"
    :title="$t('core.dashboard.widgets.presets.recent_published.title')"
  >
    <OverlayScrollbarsComponent
      element="div"
      :options="{ scrollbars: { autoHide: 'scroll' } }"
      class="h-full w-full"
      defer
    >
      <VEntityContainer>
        <VEntity v-for="post in data" :key="post.post.metadata.name">
          <template #start>
            <VEntityField
              :title="post.post.spec.title"
              :route="{
                name: 'PostEditor',
                query: { name: post.post.metadata.name },
              }"
            >
              <template #description>
                <VSpace>
                  <span class="text-xs text-gray-500">
                    {{
                      $t(
                        "core.dashboard.widgets.presets.recent_published.visits",
                        { visits: post.stats.visit || 0 }
                      )
                    }}
                  </span>
                  <span class="text-xs text-gray-500">
                    {{
                      $t(
                        "core.dashboard.widgets.presets.recent_published.comments",
                        { comments: post.stats.totalComment || 0 }
                      )
                    }}
                  </span>
                  <span class="truncate text-xs tabular-nums text-gray-500">
                    {{
                      $t(
                        "core.dashboard.widgets.presets.recent_published.publishTime",
                        {
                          publishTime: formatDatetime(
                            post.post.spec.publishTime
                          ),
                        }
                      )
                    }}
                  </span>
                </VSpace>
              </template>
              <template #extra>
                <a
                  v-if="post.post.status?.permalink"
                  target="_blank"
                  :href="post.post.status?.permalink"
                  :title="post.post.status?.permalink"
                  class="hidden text-gray-600 transition-all hover:text-gray-900 group-hover:inline-block"
                >
                  <IconExternalLinkLine class="h-3.5 w-3.5" />
                </a>
              </template>
            </VEntityField>
          </template>
        </VEntity>
      </VEntityContainer>
    </OverlayScrollbarsComponent>
  </VCard>
</template>
